# Chennai One feeds

A second, independently collected set of GTFS feeds for the same city, covering
all three modes — MTC buses, CMRL Metro and, for the first time, **Southern
Railway's Chennai suburban network**.

These sit beside the feeds already in `data/`; nothing there is modified. See
[Why these are separate files](#why-these-are-separate-files) for why they are
not merged into them.

| Feed | Routes | Trips/day | Stops | Shape points | ZIP |
|------|-------:|----------:|------:|-------------:|-----|
| [`mtc/`](mtc) — MTC buses | 970 | 48,132 | 3,393 | 1,001,893 | [`chennai-one-mtc-gtfs.zip`](../chennai-one-mtc-gtfs.zip) |
| [`cmrl/`](cmrl) — CMRL Metro | 3 | 749 | 41 | 2,453 | [`chennai-one-cmrl-gtfs.zip`](../chennai-one-cmrl-gtfs.zip) |
| [`suburban/`](suburban) — Southern Railway suburban + MRTS | 573 | 576 | 105 | 130,561 | [`chennai-one-suburban-gtfs.zip`](../chennai-one-suburban-gtfs.zip) |

Built from the Chennai One network snapshot published **17 October 2026** (app 3.6.0); the
feeds are valid to 31 December 2027 and will be refreshed when CUMTA publishes a new snapshot.

## What these add

Against the limitations this repo already documents:

| Gap noted in the root README | In these feeds |
|---|---|
| "No suburban rail — this is the biggest gap" | 573 routes with exact times and track-matched geometry |
| "MTC shapes: straight-line per route" | 1,001,893 points map-matched to the road network |
| "CMRL schedules: frequency bands, not exact departure times" | 749 trips with per-station arrival and departure times |
| "CMRL shapes: straight-line, track geometry unavailable" | 2,453 points snapped to the metro alignment |

MTC trips also carry a `service_tier` column (ORDINARY, EXECUTIVE, EXPRESS, AC,
SPECIAL, NON_AC) — the board type the operator runs the trip under. It is an
extra column, ignored by any standard GTFS reader.

MTC trips also carry three columns from the day's live roster (19 October 2026),
on the 31,322 trips whose first-stop time matched it exactly:

| column | meaning |
|---|---|
| `vehicle_number` | registration of the bus rostered to the trip that day (e.g. `K5177`) |
| `vehicle_fuel` | `electric` or `diesel` — Chennai One's `EV` flag |
| `wheelchair_accessible` | standard GTFS; `1` for low-floor buses, `0` = no information |

The roster is one day's, so the registration is not used as `block_id`: on the
published run times 9.6% of a bus's consecutive trips would overlap, which
`block_id` does not allow.

## Fares

Fares come from the Chennai One app's own fare search, which prices a stop pair in
every class at once (a search only quotes; nothing is booked). Harvested 19–20 October
2026. Files are in [`fares/`](fares); stop ids are the `stop_id`s of the matching feed.

**MTC — [`fares/mtc_fare_stages.csv`](fares/mtc_fare_stages.csv).** For each working,
every stop's fare stage counted from the working's first stop and the fare from the
first stop in each class — all 3,713 workings, 78,857 stops. Every class is a fixed
function of the Ordinary fare — true of all 75,257 quotes harvested:

| class (`service_tier`) | fare |
|---|---|
| Ordinary (`ORDINARY`), Small Bus Ordinary (`NON_AC`) | ₹(4 + stage): ₹5, 6, 7 … |
| Express (`EXPRESS`), Small Bus Express (`SPECIAL`) | ⌊1.5 × Ordinary⌋: ₹7, 9, 10, 12 … |
| Deluxe (`EXECUTIVE`) | 2 × Ordinary + 1: ₹11, 13, 15 … |
| AC (`AC`) | by Ordinary fare: ₹5–6 → 15, 7–9 → 20, 10–12 → 30, 13–16 → 40, 17–18 → 50, 19–20 → 60, 21 → 60–70, 22 → 70, 23 → 70–80, 24 → 80 |

Stages are MTC's own, set per route; they do not convert to kilometres. And a
journey that does not start at the first stop **cannot** be priced from this table:
of 120 mid-route pairs checked, only 57 cost the stage difference, the rest ₹1–6
more. The 192 measured mid-route pairs are in
[`fares/mtc_fare_pairs_midroute.csv`](fares/mtc_fare_pairs_midroute.csv); for any other
pair, apply the class rules above to a quoted Ordinary fare.

**CMRL — [`fares/cmrl_fares.csv`](fares/cmrl_fares.csv).** Every station pair on the same
line, both directions (1,122). `ordinary_adult` is the ticket fare — ₹10/20/30/40/50 by
CMRL's distance slabs (0–2, 2–5, 5–12, 12–21, 21+ km); `ordinary_app_price` is what the
app charges, 20% less for a QR ticket. Journeys that change between the Blue and Green
lines are not priced by this search and are absent.

**Suburban and MRTS — [`fares/suburban_fares.csv`](fares/suburban_fares.csv).** Every
station pair a route serves, both directions (3,906): Second and First Class, adult and
child. 854 pairs, mostly to and from Royapuram / Chennai Fort and on the Arakkonam and
Kanchipuram branches, return no fare in the app on any route that serves them and are
absent rather than guessed.

## Source and method

Collected from the **Chennai One** app, the CUMTA-backed multimodal app that
carries all three operators, so bus, metro and suburban schedules come from one
consistent source. Timetables are per stop, so each trip's times are the
operator's own, not interpolated.

- **Stops** are deduplicated. One physical stop appears in the source many times
  under near-identical names and slightly different coordinates — Kilambakkam
  terminus alone has 12 records spread over ~110 m. Records are clustered by
  normalised name within 200 m, then by shared distinctive token within 75 m,
  then unconditionally within 5 m. That takes 6,936 MTC records to 3,393 stops.
- **Bus shapes** are produced by routing each working's whole stop sequence in a
  single OSRM call against the OSM road network and keeping the returned
  polyline. Routing stop-pair by stop-pair and re-stitching is what produces the
  looping geometry seen in other Chennai feeds.
- **Rail shapes** are produced by Dijkstra routing over OSM rail geometry, with
  separate graphs per mode so Metro cannot leak onto suburban track.
- **Shape geometry is simplified** with Douglas-Peucker at 1 m, below the
  positional accuracy of the centrelines the routes were matched against. This
  drops 54% of bus vertices without moving a line off its road.
- **Bus stop times are derived from the first stop.** Every stop's time is the
  first-stop time plus the route's cumulative stop-to-stop run times, with a 15 s
  dwell — identical to the per-stop timetables on all 6,351 trip-stops sampled.
- **Times are unwrapped across midnight** — a trip leaving at 22:25 and arriving
  after midnight counts on past 24:00:00 rather than resetting to 00:xx.
- **Trips are attributed by trip id.** A trip appears in the timetable of every
  route whose stops it passes — one suburban train shows up under 558 route
  entries — and its id names the route that runs it. Keying on (route, trip id)
  instead multiplies 52,000 real trips into 973,000.
- Routes with nothing scheduled on them, trips with fewer than two stop times,
  and stops nothing calls at are dropped rather than published broken.
- **Placeholder stations are dropped.** CMRL files its Phase-2 construction
  stations under names like `[TEST]-0417` — ten of them along the Poonamallee
  alignment, with real coordinates and a full timetable. They are not stations
  anyone can board at, and publishing them would put an unbuilt line in the feed
  as though it were in service.

## Why these are separate files

The two collections are not mergeable row for row:

|  | this repo's `data/mtc` | `chennai-one/mtc` |
|---|---|---|
| route rows | 4,611 | 970 services over 3,713 workings |
| distinct `route_short_name` | 2,305 (`101 CT1`, `101 CT11`, … counted separately) | 970 |
| stops | 5,580 | 3,393 deduplicated |
| stop names in common | 1,512 | 1,512 |

The route ids come from different collections, so there is no key to join on and
grafting these shapes onto the existing route ids would be guesswork. Publishing
them side by side lets both be used and compared. If you would rather these
replace or merge into the primary feeds, say so on the PR and we will do that
work.

## Licence

Published under ODbL, the same licence as the rest of this repo.
