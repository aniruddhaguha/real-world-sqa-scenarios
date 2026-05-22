# Scenario 05: Ride Starts Automatically Without Driver Action

## Scenario

In a ride-sharing app, some rides start automatically without the driver clicking the **"Start Trip"** button. This only happens when **GPS signal is weak** or when the **driver switches between mobile data and Wi-Fi**.

---

## Assumptions

- The "Start Trip" action is an explicit button press that sends a request to the backend.
- The app uses GPS location for ride tracking and background location services.
- Trip start logic may also have geofence-based or proximity-based auto-triggers as a fallback.
- When switching between mobile data and Wi-Fi, the app may reconnect and replay buffered events.
- The app runs background services to maintain location updates even when not in foreground.
- The issue is intermittent — not reproducible 100% of the time.

---

## Test Ideas

### GPS Signal Testing
- Test the app in a GPS-weak environment (underground parking, inside buildings, tunnels).
- Simulate GPS signal degradation using mock location tools or device settings.
- Observe if any auto-start trigger fires when GPS accuracy drops below a threshold.
- Check if the app has a geofence trigger (e.g., "driver is within X meters of pickup") that could inadvertently trigger trip start.

### Network Switch Testing
- Simulate switching from Wi-Fi to mobile data while a ride is in "Accepted" state — observe app behavior.
- Check if pending API requests (including "Start Trip") are queued and replayed when the network reconnects.
- Use a proxy (Charles, Proxyman) to inspect outgoing API calls during network switch — look for duplicate or unexpected "start trip" calls.
- Simulate multiple rapid network switches and confirm the app's state machine remains consistent.

### Background Service Testing
- Test app behavior when the driver app is in the background during network reconnect.
- Check if any background location update inadvertently triggers a trip-start event.
- Verify that background service events are properly gated by explicit user action checks.
- Test the app after the OS kills and restarts background services (low memory scenario).

### State Machine Validation
- Review and test all allowed state transitions: Accepted → En Route → Started → Completed.
- Verify that "Trip Started" state can only be entered via the explicit driver button action.
- Confirm no automated process (geofence, location event, reconnection) can push the trip into "Started" state without driver confirmation.

---

## Edge Cases

- Driver presses "Start Trip" button, network is unavailable at that moment — request queued. Network reconnects. Request fires. But the driver pressed the button again — two "start" events sent.
- GPS jumps (jitter) causes the app to think driver has reached pickup point, triggering an auto-start.
- WebSocket or long-polling connection drops during network switch and reconnects — server replays last event which was "Trip Accepted" and the client misinterprets it as "Start Trip."
- OS-level location permission changes during a ride (user revokes then re-grants) cause unexpected callbacks.
- App crashes and restarts mid-ride — on restart, it queries last known state from server and misapplies it.
- Timer-based fallback: if "Start Trip" is not pressed within N minutes, the system auto-starts (poor design, but possible).
- Stale event in a message queue processed after network reconnect triggers start.

---

## Risks

- **Legal/liability risk:** A trip started without the driver's knowledge may charge the passenger unfairly or create a false record.
- **Driver trust risk:** Drivers unaware their trip has started may not be at the pickup point — leads to complaints.
- **Safety risk:** Auto-started trips may result in passengers being charged for distance the driver didn't actually travel with them.
- **Data integrity risk:** Trip records show incorrect start times and locations.
- **Regulatory risk:** In regulated markets, incorrect trip records can lead to compliance violations.
- **Reproducibility risk:** Intermittent GPS and network events make this bug hard to reproduce in controlled environments.
