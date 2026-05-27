# Scenario 06: Videos Show 100% Complete but Certificate Remains Locked

## Scenario
    
An online learning platform tracks video progress and unlocks certificates when **100% of the course is completed.** Many users report that their videos show **fully watched**, but certificates remain locked even after refreshing and re-logging.

---

## Assumptions

- Video progress is tracked via events sent from the video player (e.g., every N seconds, or at completion).
- Certificate unlocking is triggered by a backend job or event that evaluates total course completion.
- "100% watched" is stored in a progress table per user per video.
- The certificate unlock condition checks multiple factors: all videos watched, quizzes passed, minimum watch time, etc.
- The UI reads certificate status from a separate endpoint/service.
- Some users may have skipped content or used seek to reach the end without fully watching.

---

## Test Ideas

### Event Tracking Validation
- Monitor network calls from the video player — confirm progress events (e.g., `video_progress`, `video_complete`) are being sent.
- Verify that the backend receives and stores these events correctly.
- Check if events sent when the browser/app is in the background are dropped or delayed.
- Test what happens if a user closes the browser mid-video — does the progress event fire before close?

### Progress Calculation Testing
- Query the database for a user with "100% shown" in UI — verify the raw progress data matches.
- Test the completion calculation logic: does it use total watch time, last position, or event-based completion?
- Check if skipping to the end of the video counts as 100% — or if the platform requires a minimum watch ratio.
- Test videos of different lengths — confirm calculation works uniformly.

### Certificate Trigger Testing
- Manually set a user's progress to 100% in the database and trigger the certificate check — does it unlock?
- Test the completion trigger: is it event-driven (fires immediately when last video is completed) or batch (runs every N minutes/hours)?
- Confirm the trigger handles all required conditions: videos, quizzes, assignments, etc.
- Simulate a scenario where one of several required conditions is met but others are not — does the UI mislead the user by showing all videos as complete?

### Cross-Service Consistency Testing
- Check if the progress service and certificate service share the same data source or have their own copies.
- Verify that progress updates are propagated to the certificate service in real time or near real time.
- Test what happens when the certificate service is temporarily down — is progress lost or queued?

---

## Edge Cases

- User watches video at 1.5x or 2x speed — does the platform track time-based or position-based completion?
- User watches video on mobile and desktop simultaneously — which progress record wins?
- Video player sends a `complete` event but network drops before it reaches the server.
- User re-watches a completed video — does it reset progress or maintain 100%?
- Course is updated (new video added) after user completed it — does the completion status reset?
- Certificate check runs before the last progress event is saved (race condition between trigger and DB write).
- User completes the course in multiple sessions across different devices — progress merge logic may be incorrect.

---

## Risks

- **User frustration risk:** Users who legitimately completed the course cannot obtain their certificate — leads to complaints and support tickets.
- **Reputation risk:** Certificates are a key value proposition of learning platforms — failure to deliver them damages trust.
- **Revenue risk:** Users may request refunds if they cannot access the certificate they paid for.
- **Data integrity risk:** Progress data shown in UI does not match what the backend uses for certificate evaluation.
- **Scalability risk:** If certificate unlocking is batch-based, delays worsen under high user load.
- **Compliance risk:** For professional certifications, incorrect issuance or denial of certificates may have regulatory implications.

