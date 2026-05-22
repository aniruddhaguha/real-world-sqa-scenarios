# Scenario 08: Push Notifications Arrive Hours Late or Not at All

## Scenario

Push notifications arrive immediately for some users, but for others they arrive **hours later or not at all.** This issue is inconsistent across devices, OS versions, and internet speeds.

---

## Assumptions

- Push notifications are sent via FCM (Firebase Cloud Messaging) for Android and APNs for iOS.
- The backend publishes notification events to a message queue (e.g., Kafka, SQS).
- A notification service consumes from the queue and calls FCM/APNs.
- Device tokens are stored per user and must be current.
- Some users have battery optimization or background data restriction enabled.
- The issue is not 100% reproducible and varies by device state.

---

## Test Ideas

### Backend Queue Testing
- Monitor the message queue for lag — check if notification events are being produced but consumed slowly.
- Check if the notification service is under-provisioned and falling behind during peak hours.
- Verify dead-letter queues for failed notification deliveries — how many are piling up?
- Test notification delivery time end-to-end from event trigger to device receipt under various load levels.

### Device State Testing
- Test notifications on a device with battery optimization enabled vs. disabled.
- Test when the app is in foreground, background, and fully closed (force stopped).
- Test on devices with Background App Refresh disabled (iOS) and Background data restricted (Android).
- Test notifications in Doze Mode on Android — FCM high-priority messages should bypass Doze, verify this is configured.
- Test on different Android versions (8, 10, 12, 13, 14) and iOS versions — handling differs significantly.

### OS & Background Restriction Testing
- Test with "Do Not Disturb" mode enabled.
- Test on manufacturer-customized Android ROMs (Xiaomi MIUI, Samsung One UI, Huawei EMUI) known for aggressive background killing.
- Verify that the app requests and receives proper notification permissions on install.
- Test on a freshly installed app vs. an app that has been on the device for months.

### Notification Service & FCM/APNs Testing
- Verify that FCM/APNs tokens are refreshed when the app is reinstalled or updated.
- Check if stale/expired tokens are cleaned up — sending to expired tokens wastes quota and indicates user is not reachable.
- Test notification priority: ensure time-sensitive notifications use HIGH priority in FCM payload.
- Verify APNs certificate or key is not expired — expired credentials cause silent delivery failures.
- Monitor FCM/APNs response codes for errors (e.g., `NotRegistered`, `InvalidRegistration`).

---

## Edge Cases

- User reinstalls the app — old token becomes invalid but is still stored in the backend.
- Device is offline for several hours — FCM/APNs queues the notification but drops it after TTL expires.
- User changes phone — old device token is still in the database; new device has a different token.
- App is force-stopped by the user on Android — FCM may not deliver until the app is manually reopened.
- Notification quota exceeded on FCM — batch deliveries are throttled.
- Clock skew on the notification service causes TTL to be calculated incorrectly (notification expires too early).
- User has multiple devices logged in — notification delivered to only some devices or duplicated.

---

## Risks

- **User experience risk:** Late or missing notifications cause users to miss time-sensitive information (OTPs, order updates, alerts).
- **Business risk:** Failed promotional notifications reduce campaign effectiveness and revenue.
- **Security risk:** Delayed OTP notifications may cause authentication failures or security confusion.
- **Retention risk:** Users who consistently miss notifications may disengage from the app.
- **Reliability risk:** Inconsistency across devices makes the issue hard to reproduce and debug.
- **Quota risk:** Sending to millions of stale tokens wastes FCM quota and may cause throttling for valid deliveries.
