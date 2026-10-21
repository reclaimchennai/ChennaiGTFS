# Chennai One feeds

A second, independently collected set of GTFS feeds for the same city, covering
all three modes — MTC buses, CMRL Metro and, for the first time, **Southern
Railway's Chennai suburban network**.

These sit beside the feeds already in `data/`; nothing there is modified. See
[Why these are separate files](#why-these-are-separate-files) for why they are
not merged into them.

| Feed | Routes | Trips/day | Stops | Shape points | ZIP |
|------|-------:|----------:|------:|-------------:|-----|
| [`mtc/`](mtc) — MTC buses | 970 | 48,132 | 3,393 | 889,835 | [`chennai-one-mtc-gtfs.zip`](../chennai-one-mtc-gtfs.zip) |
| [`cmrl/`](cmrl) — CMRL Metro | 3 | 749 | 41 | 2,453 | [`chennai-one-cmrl-gtfs.zip`](../chennai-one-cmrl-gtfs.zip) |
| [`suburban/`](suburban) — Southern Railway suburban + MRTS | 753 | 576 Mon–Sat · 591 Sun | 105 | 171,110 | [`chennai-one-suburban-gtfs.zip`](../chennai-one-suburban-gtfs.zip) |

Built from the Chennai One network snapshot published **17 October 2026** (app 3.6.0); the
feeds are valid to 31 December 2027 and will be refreshed when CUMTA publishes a new snapshot.

## What these add

Against the limitations this repo already documents:

| Gap noted in the root README | In these feeds |
|---|---|
| "No suburban rail — this is the biggest gap" | 753 train services with exact times, weekday and Sunday service days, and track-matched geometry |
| "MTC shapes: straight-line per route" | 889,835 points map-matched to the road network |
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

**MTC's own fare table — [`fares/mtc_official_stage_fares.csv`](fares/mtc_official_stage_fares.csv)**
(and `.json`). MTC publishes stage-wise fares for one adult in every service class at
[mtcbus.tn.gov.in/Home/fares](https://mtcbus.tn.gov.in/Home/fares), in force since
29 January 2018 (G.O. (Ms.) No. 48, 28 January 2018); read 21 October 2026. A stage is
about 2 km, and a ₹1 cess is collected per passenger in every class.

| class | stage 1 → 30 |
|---|---|
| Ordinary, Pink | ₹5, rising ₹1 a stage to ₹15 at stage 11, then ₹1 every two stages to ₹24 |
| Express, Small | ₹7 → ₹35 (about 1.5 × Ordinary) |
| Deluxe, Night, E/V Deluxe, Chennai Ula Vintage | ₹11 → ₹49 (2 × Ordinary + 1) |
| E/V A/C, Chennai Ula A/C | ₹15, 15, 20, 20, 20, 30 … in steps of ₹10 to ₹80 |
| Premium | ₹50, 75, 100, 125, 150 for stages 1–5 |

Chennai Ula also sells a one-day hop-on pass, ₹50 (Vintage) and ₹100 (A/C).

**Fares per stop — [`fares/mtc_fare_stages.csv`](fares/mtc_fare_stages.csv).** For each
of the 3,713 workings, every stop's fare from the working's first stop in four classes
(`ORDINARY`, `EXPRESS`, `EXECUTIVE` = Deluxe, `AC`), as quoted by the Chennai One app's
fare search (harvested 19–20 October 2026; a search only quotes, nothing is booked), and
the stop's official fare stage from the first stop, `fare_stage_min`–`fare_stage_max`,
found by matching all four quoted fares against MTC's table. Above stage 11 an
Ordinary fare covers two stages, so a stop gets one exact stage where the classes tell
them apart (61,726 of 78,857 stops) and a two-stage range otherwise. The app's quotes
agree with MTC's table in every class at every stage, except at the top Ordinary fare
(₹24), where the app quotes Express ₹36 against the table's ₹35 (10 stops; their
stage is left blank). Stages are set per route by MTC and do not convert exactly to
kilometres.

**Mid-route quotes — [`fares/mtc_fare_pairs_midroute.csv`](fares/mtc_fare_pairs_midroute.csv).**
192 quotes for journeys that do not start at a working's first stop. Do not use them
as fares: of the 138 that can be checked, 130 equal the fare from the working's
*first* stop to the alighting stop, as if the rider had boarded at the start. The
fare for such a journey is MTC's table at the number of stages travelled.

**CMRL — [`fares/cmrl_fares.csv`](fares/cmrl_fares.csv).** Every station pair on the same
line, both directions (1,122). `ordinary_adult` is the ticket fare — ₹10/20/30/40/50 by
CMRL's distance slabs (0–2, 2–5, 5–12, 12–21, 21+ km); `ordinary_app_price` is what the
app charges, 20% less for a QR ticket. Journeys that change between the Blue and Green
lines are not priced by the app's search; their 600 pairs come from CUMTA's published
fare rules (`source` = `cumta`; the app price is the same 20% off). CUMTA's fares agree
with every pair the app priced. The CMRL feed also carries them as standard
`fare_attributes.txt` / `fare_rules.txt`, with each station's CMRL code as its `zone_id`.

**Suburban and MRTS — [`fares/suburban_fares.csv`](fares/suburban_fares.csv).** Every
station pair a route serves, both directions (3,906): Second and First Class, adult and
child. 854 pairs, mostly to and from Royapuram / Chennai Fort and on the Arakkonam and
Kanchipuram branches, return no fare in the app on any route that serves them and are
absent rather than guessed.

## From CUMTA's published GTFS

CUMTA publishes static GTFS for all three modes at
[test.cumta.org/datasets](https://test.cumta.org/datasets) (Government Open Data
Licence – India; files dated 2 February 2026, timetables from October 2025). It comes
from the same backend as the Chennai One app, about a year older, so its timetables
are not used where ours exist. These are taken from it
(`enrich_from_cumta.py` in the build pipeline):

- **Station structure (CMRL and suburban).** Every stop keeps its `stop_id` and is
  still the boarding point `stop_times` use; it now has a parent station
  (`STN_<code>`, `location_type` 1) and its `stop_code` is the station code (`MSB`,
  `TBM`; CMRL's `SCC`, `SAL` …). 212 CMRL and 179 suburban entrances are added as
  `location_type` 2 (`ENT_…`), CMRL's with their own descriptions ("A2 entrance
  physically challenged ramp (near by lift)"), each joined to the platform by a
  walkway in `pathways.txt`. All 41 CMRL and 104 suburban stations matched by
  position and name.
- **CMRL fares** — as above.
- **Suburban service days.** Chennai One publishes one every-day calendar and no
  Sunday trains at all. CUMTA splits trains into daily, Monday–Saturday and
  Sunday-only. The 540 trains in both feeds (same trip id = same train service) take
  CUMTA's days: 375 `Daily`, 165 `MonSat`. The 36 trains CUMTA does not have stay on
  `Regular` (every day, days unconfirmed). CUMTA's 180 Sunday-only trains are added
  with its times, on service `Sunday`; 169 have a shape cut from a longer train's on
  the same track, 11 have none. Of the trains in both feeds, 143 have moved their
  departure since CUMTA's timetable, so Sunday times may be a little out of date.
- **Not used:** CUMTA's MTC bus feed (1,634 of its 7,128 stops sit on one placeholder
  coordinate, 12,800 trips repeat, it has no shapes), its per-train platform numbers
  (a year old), and its CMRL timetable (calendar ended 2021).

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
- **Bus shapes follow the bus's own road.** Routing stop to stop takes the shortest
  roads, and at junctions OSRM attached some stops to the side road beside the bus's
  road, dragging the line through interior streets (121A and 170CET at the Madhavaram
  Milk Colony gate, reported from the ground). Stops are therefore routed from their
  position on the operator's own path (the app's route `waypoints`), with that path's
  heading as an OSRM bearing, via points along it; legs that still loop (a stop on the
  far carriageway) are re-laid on the operator path. Median shape length against the
  operator path went from +18% to +3%; about 9,200 km of detours removed in all.
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

Published under ODbL, the same licence as the rest of this repo. Station codes, entrances,
CMRL fare rules, suburban service days and Sunday trains are from CUMTA's datasets,
used under the Government Open Data Licence – India: source CUMTA (Chennai Unified
Metropolitan Transport Authority).
