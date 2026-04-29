# Places Coordinate Deep Link

This document describes the custom deep link support that was added for opening the modified Wikipedia app directly on the `Places` tab using coordinates from another app.

## Summary

The app now supports a custom URL scheme for opening `Places` with:

- a target coordinate
- an optional article title used as the preferred selected result

Example:

```text
wikipedia-dev://places?lat=52.3547498&lon=4.8339215&title=Amsterdam
```

## What Changed

### `Wikipedia/Wikipedia-Info.plist`

Added a custom URL scheme:

- `wikipedia-dev`

This lets another iOS app launch the modified Wikipedia app without conflicting with the stock `wikipedia://` scheme.

### `Wikipedia/Code/SceneDelegate.swift`

Added a custom deep link flow for:

- parsing `wikipedia-dev://places`
- extracting `lat`, `lon`, and `title`
- deferring processing until the app UI is ready
- switching to the `Places` tab safely on cold start and normal app launches

Implementation notes:

- deep link data is stored in `pendingPlacesDeepLink`
- processing happens from `sceneDidBecomeActive(_:)`
- the app resets the `Places` tab to a root map state before applying the deep link

### `Wikipedia/Code/PlacesViewController.swift`

Added:

- `showCoordinates(_ coordinate: CLLocationCoordinate2D, title: String?)`

This method:

- switches the screen to map mode
- centers the map around the incoming coordinate
- starts a `Places` search for the surrounding region
- creates a `MWKSearchResult` from the incoming `title` so the intended article can be selected instead of simply choosing the nearest point of interest

## Deep Link Format

Supported parameters:

- `lat`: latitude as `Double`
- `lon`: longitude as `Double`
- `title`: article title to prioritize in the search results

Example:

```text
wikipedia-dev://places?lat=19.0823998&lon=72.8111468&title=Mumbai
```

## Behavior

When the deep link is opened:

1. The modified Wikipedia app launches.
2. The URL is parsed in `SceneDelegate`.
3. The app waits until the main UI is available.
4. The app switches to the `Places` tab.
5. The map centers on the provided coordinates.
6. The article named by `title` is preferred when matching the selected place.

## Testing

Example Simulator command:

```bash
xcrun simctl openurl booted "wikipedia-dev://places?lat=52.3547498&lon=4.8339215&title=Amsterdam"
```

Suggested manual checks:

- cold start via deep link
- app already running in foreground
- app already running in background
- app currently deep inside the `Places` navigation stack

## Notes

- This is custom behavior for the modified app and is not part of the stock Wikipedia iOS app.
- The custom flow is intentionally separate from the existing `wikipedia://` handling.
