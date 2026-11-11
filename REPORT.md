# Jalan — Milestone 5: Directions Planner & Navigation Experience

## Overview

Milestone 5 builds a directions planner and approximate, GPS-assisted route-guidance mode on top of Jalan's existing vanilla JavaScript, MapLibre, Nominatim, and OSRM implementation. It remains a prototype and does not provide production-grade or turn-by-turn live navigation.

## Features implemented

- Search results retain their existing selection behavior and now include a **Directions** action that selects the destination and opens the planner.
- The planner supports a selected origin and destination, the existing **Use my location** flow, origin/destination search using the existing search box, swapping endpoints, clearing a route, closing the planner, and a travel-mode selector. Driving is currently the only available mode; walking and transit are explicitly unavailable.
- OSRM-provided routes appear as selectable cards with estimated time, distance, and route description when available. Selecting an alternative updates map emphasis, route bounds, summary, and directions. No traffic or fastest-route claims are made.
- **Start navigation** opens a compact, high-contrast guidance panel and enables map following. It reuses an existing GPS watch or starts one on the user's explicit action.
- Approximate route progress projects accepted GPS positions onto the selected route geometry to estimate the next manoeuvre, distance to that manoeuvre, remaining route distance, and remaining time.
- GPS readings farther than `max(75 m, 2 × reported accuracy)` from the selected route trigger an off-route warning. Approximate arrival is detected within 60 m of the selected destination and is announced only once per navigation session. Neither case triggers rerouting.
- **End** restores the route planner and route preview. Tracking started before navigation remains active and its earlier follow-mode preference is restored; tracking started by navigation is stopped.
- Manual origin/destination search, swapping, and the navigation panel preserve the existing route/search markers and route layers until route changes or the user clears the route.
- Desktop directions appear on the left side of the map; the mobile planner and navigation panel are sized for a 390px viewport. Essential map controls are raised above the mobile navigation panel.
- GPS coordinates remain in memory. They are not written to storage or sent to external services except when the user explicitly calculates a route using the chosen origin and destination.

## Files modified

- `index.html` — search-result Directions actions, endpoint search/swap controls, mode selector, route cards, Start button, and navigation guidance panel.
- `style.css` — desktop directions panel, route cards, responsive mobile layout, and high-contrast navigation panel.
- `script.js` — endpoint selection and swapping, route-mode handling, navigation lifecycle, watcher reuse, route-progress projection, off-route and arrival detection, and cleanup.
- `REPORT.md` — milestone features, architecture, verification, instructions, and limitations.

## Architecture

The existing search, route calculation, alternative rendering, and GPS watcher remain the shared services. A single `searchTarget` chooses which planner endpoint the existing Nominatim search populates. `currentLocation` remains the route origin; the live `trackedPosition` is separate and is used only for the moving marker, map follow, and navigation progress. Navigation uses the existing `calculatedRoutes` and selected route geometry without making additional routing requests. No new dependencies or frameworks were added.

## Tests and results

- `node --check script.js` and `node --check api-config.js` pass.
- HTML checks found no duplicate IDs or buttons missing an explicit button type.
- Browser tests with mocked Nominatim, OSRM, and GPS verified the result-to-Directions workflow, manual origin search, endpoint swapping, route calculation, alternative selection, and route-summary updates.
- Mocked GPS tests verified watcher reuse, follow-mode entry/restoration, GPS-driven manoeuvre and remaining-distance updates, off-route warning, one-time approximate-arrival state, and ending navigation while preserving the route and a pre-existing tracker.
- Mocked permission-denial and timeout checks verified understandable errors and cleanup. Earlier mocked invalid/inaccurate GPS checks also verified those readings are rejected.
- At 390×844, the navigation panel and essential map controls were checked for overlap. Real-device GPS, sensor accuracy, and real permission behavior were not tested.
- All GPS, search, and route responses used in UI behavior tests were mocked. No real OSRM route was used to verify navigation progress. Public map tiles loaded in the browser; this is not a GPS or route-service verification.
- No project test runner, automated visual-regression suite, or ESLint configuration is included.

## Manual testing

Serve this folder from `localhost` or HTTPS, then:

1. Search for a Malaysian place. Select **Directions** on a result and confirm the planner opens with that destination.
2. Set an origin with **Search**, or select **Use my location**. Search and set a destination, swap endpoints, then calculate a driving route.
3. If OSRM returns alternatives, select each card and verify the selected line, bounds, distance, duration, and instructions update. Availability depends on OSRM.
4. Select **Start navigation** and grant GPS permission. Confirm the next manoeuvre and remaining estimates update with new positions. Manually move the map and re-enable follow if needed.
5. In a local test harness, feed points along the route, beyond the off-route threshold, and within 60 m of the destination. Confirm the off-route warning, approximate arrival, and no automatic rerouting.
6. End navigation. Confirm the selected route preview is preserved. If tracking was already active before navigation, it should remain active; if navigation started it, the watcher should stop.
7. Test permission denial, timeout, invalid coordinates, and low-accuracy fixes with mocked geolocation. Repeat planner and navigation layout checks at 390px and a desktop viewport.

## Known limitations and excluded features

- This is approximate route guidance, not production-grade or precise turn-by-turn live navigation. Route progress is based on straight-line projection to OSRM geometry and device GPS fixes.
- No automatic rerouting, voice guidance, live traffic, background tracking, speed-camera alerts, or community reports are implemented.
- Walking and transit routing are unavailable; only OSRM driving routes are supported.
- Route estimates do not account for live traffic. Public OSRM, Nominatim, and map services have independent availability and usage limits.
- GPS availability, permission UX, accuracy, and heading depend on the browser and device. Real-device testing remains necessary.
