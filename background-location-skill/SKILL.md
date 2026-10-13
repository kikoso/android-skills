---
name: android-background-location-skill
description: Enforces reliable, battery-aware and privacy-safe background location tracking on Android. Use when implementing or reviewing location tracking, foreground services, location permissions, geofencing, activity recognition, or anything that must keep running while the app is in the background.
---

# Android Gravity Skill: Background Location

## Description
This skill enforces correct, resilient and policy-compliant background location tracking. Tracking that silently stops in the background is a functional bug (a missed check-out, a wrong trip, a wrong fare), so every change touching location is treated as high risk.

## Instructions
When activated, you MUST adhere to the following rules:

### 1. Permissions Flow
- **Foreground First:** Request `ACCESS_FINE_LOCATION` and/or `ACCESS_COARSE_LOCATION` first. Request `ACCESS_BACKGROUND_LOCATION` only afterwards, as a separate step (Android 10+). On Android 11+ it cannot be granted from the runtime dialog: explain why, then send the user to the app's settings page ("Allow all the time").
- **Handle Approximate Only:** Since Android 12 the user may grant only `ACCESS_COARSE_LOCATION`. Detect it and degrade explicitly (explain what no longer works and offer to upgrade to precise) instead of assuming fine location.
- **Notifications Permission:** On Android 13+ request `POST_NOTIFICATIONS`. Without it the foreground service notification is hidden, and the user loses the visible signal that tracking is active.
- **Activity Recognition:** If the Activity Recognition API is used, request `ACTIVITY_RECOGNITION` (Android 10+) as its own runtime permission.
- **Re-check on Every Start:** Permissions can be revoked at any time (including auto-reset for unused apps). Check the current state before every tracking session; never cache a past grant.

### 2. Foreground Service
- **Use a Foreground Service for Continuous Tracking:** Ongoing tracking (e.g. during a trip) MUST run in a foreground service with `android:foregroundServiceType="location"`, declare `FOREGROUND_SERVICE` and `FOREGROUND_SERVICE_LOCATION` (required on Android 14+), and pass the type to `startForeground(...)` (`ServiceCompat.startForeground`).
- **Start It From the Foreground:** Since Android 12, apps generally cannot start a foreground service from the background (`ForegroundServiceStartNotAllowedException`). Start tracking from a user action or a visible UI. Without background location permission, a location FGS only receives location if it was started while the app was in the foreground.
- **Meaningful Notification:** The notification must state what is being tracked and offer a clear way to stop it (e.g. a "Check out" action). Never use an empty or misleading notification.
- **Stop Deterministically:** Stop the service and remove location updates on every exit path (check-out, error, permission revoked, session timeout). Leaked location requests drain the battery and break user trust.

### 3. Choosing the Right API
- **Fused Location Provider:** Use `FusedLocationProviderClient` with `LocationRequest.Builder`. Choose `Priority` and intervals per state (e.g. high accuracy while moving, balanced or low power when stationary). Never default to high accuracy with a 1 s interval.
- **Activity Recognition & Geofencing for Triggers:** Use the Activity Recognition Transition API or geofences to *detect* when tracking is needed and to adapt the sampling rate. Respect the geofence limit (100 per app per device user) and expect background geofence alerts to be delayed by minutes.
- **WorkManager Is Not for Tracking:** Use WorkManager only for deferrable work (uploads, sync, cleanup). Periodic work runs at most every 15 minutes and is deferred under Doze, so it cannot replace a foreground service.
- **No Polling Loops:** Never keep the device awake with `while` loops, `Handler` polling or wake locks to fetch location.

### 4. Doze, Standby and OEM Restrictions
- **Design for Doze and App Standby Buckets:** Assume network access and alarms are deferred when the device is idle. Buffer location points locally (Room) and upload them when connectivity and constraints allow.
- **Battery Optimization Exemptions:** Only request `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS` if the use case qualifies under Google Play policy, and document why. Prefer guiding the user to the settings screen with an explanation.
- **OEM Battery Killers:** Some manufacturers (e.g. Xiaomi, Huawei, Samsung, OnePlus) kill background apps aggressively. Detect unexpected service death (e.g. a stale "last location" timestamp), show in-app guidance (see dontkillmyapp.com), and log the event (without coordinates) for monitoring.
- **Process Death:** Persist the tracking session state (session id, start time, last known point) so the app can recover or close the session correctly after process death or reboot.

### 5. Data Quality
- **Filter Bad Fixes:** Drop or down-weight points with poor `accuracy`, impossible speeds (jumps), stale timestamps, or `isMock` set (`isFromMockProvider` before API 31), according to the product rules.
- **Use Monotonic Time for Intervals:** Use `elapsedRealtimeNanos` to compute durations between fixes; wall-clock time can jump (time zone changes, DST, manual changes).
- **Never Invent Data:** If tracking had a gap, record and surface the gap. Do not interpolate silently in a way that changes what the user is charged.

### 6. Privacy and Policy
- **Prominent Disclosure:** Before requesting background location, show an in-app disclosure explaining what is collected, why, and when. Google Play requires it, and a Play Console declaration for background location.
- **Data Minimization:** Collect location only during an active session, keep raw points only as long as needed, and prefer storing derived data (stops, route segments) over raw traces.
- **No Coordinates in Logs:** Never write coordinates, addresses or trip traces to Logcat, analytics or crash reports. Log state transitions and counters instead.

### 7. Verification
After changing location code, verify the background behavior, not just the happy path:
- Force Doze: `adb shell dumpsys deviceidle force-idle` (undo with `adb shell dumpsys deviceidle unforce`).
- Change standby bucket: `adb shell am set-standby-bucket <package> rare`.
- Revoke permissions while tracking: `adb shell pm revoke <package> android.permission.ACCESS_BACKGROUND_LOCATION`.
- Kill the process mid-session: `adb shell am kill <package>`, then confirm the session recovers or closes correctly.
- Replay routes with mock locations or the emulator's route playback, including GPS loss and accuracy jumps.
- Report which of these checks were actually run. Never claim background behavior works if only foreground testing was done.

## Triggers
Activate this skill whenever:
- Adding or changing location permissions, `LocationRequest`s, geofences or Activity Recognition.
- Creating or modifying a `Service`, especially a foreground service, or `foregroundServiceType` in `AndroidManifest.xml`.
- Implementing check-in/check-out, trip detection, or any feature that must keep working when the app is in the background.
- Reviewing a diff that touches battery optimization, wake locks, WorkManager scheduling for tracking, or location logging.
- The user reports tracking stopping, missed check-outs, high battery drain, or device-specific (OEM) issues.
