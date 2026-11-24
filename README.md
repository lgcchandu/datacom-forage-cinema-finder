# Cinema Finder: MapLibre Snapping Bug Case Study

A learning case study from Datacom's Software Development Job Simulation on Forage, completed on November 24, 2026.

This was a self-paced job simulation, not employment or an internship at Datacom.
The application and starter code were provided by the simulation. Analysis, the proposed fix, and regression verification were completed with ChatGPT assistance.

## Application
Cinema Finder helps users browse cinemas in Australia and New Zealand. The review task involved summarising customer feedback into bugs and proposed features. The implementation task involved investigating and fixing one bug.

## Investigation
The cinema list dispatches a `map.snapTo` event with named `lat` and `lng` fields. The MapLibre listener used these values in the wrong order when calling `flyTo`.

MapLibre expects longitude before latitude. The application's Leaflet implementation uses latitude before longitude; the two APIs must not be treated interchangeably.

For example, a cinema at latitude -33.6520892 and longitude 115.3463871 was passed with 115.3463871 in the latitude position, outside latitude's valid range.

## Focused correction
File: `src/components/Map/MaplibreMap.jsx`

| Parameter | Original | Corrected |
| --- | --- | --- |
| Camera centre | `[lat, lng]` | `[lng, lat]` |

Only the MapLibre camera coordinate ordering was corrected. Cinema information popups and other reported issues were outside this fix.

## Evidence and limits

| Check | Result |
| --- | --- |
| Original incorrect order recreated in an isolated handler check | Error path reproduced |
| Corrected camera arguments checked against supplied cinema records | 549 records passed |
| Existing project tests | 3 tests passed across 2 suites |
| Full production build under Node 24.22.3 | Blocked by legacy Webpack/OpenSSL incompatibility |
| End-to-end map rendering and camera movement | Not verified |

The regression harness executes the source event-listener function with mocked map and hook boundaries. It checks arguments and retained error reporting; it does not mount React, render WebGL, or load map tiles. The 549 records are data cases, not 549 browser tests.

Browser developer-console inspection did not yield a usable application error trace because the sandbox preview refused to connect. The root cause was established through source tracing and isolated checks.

## Lessons
- Coordinate conventions differ across mapping libraries.
- A small code change still needs a failing example and verification.
- Report customer claims separately from directly observed behavior.
- Passing isolated tests does not establish that the full application works.

## Remaining work
- Run the application in a compatible development environment.
- Verify cinema snapping in MapLibre and check the console.
- Confirm Leaflet behavior remains intact.
- Capture reproducible before/after browser evidence.

## Attribution
- Simulation: Datacom Software Development Job Simulation, hosted by Forage.
- Starter source: https://codesandbox.io/p/sandbox/cinema-finder-poc-wgejtg
- Coordinate reference: https://maplibre.org/maplibre-gl-js/docs/API/type-aliases/LngLatLike/
- Participant: Lohit Chandu (https://github.com/lgcchandu).

This repository contains an explanatory write-up. The complete third-party starter code is not included; its redistribution permissions have not been established.
