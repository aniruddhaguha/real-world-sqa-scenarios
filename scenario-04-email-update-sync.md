# Scenario 04: Email Update Shows in UI but Old Email Receives System Emails

## Scenario

A user updates their email address in their profile successfully. The UI shows the new email everywhere. However, **password reset links and promotional emails** are still being sent to the old email address days later.

---

## Assumptions

- The user profile service, authentication service, and email/marketing service are separate systems.
- Email update is persisted in the primary user database.
- The authentication service has its own store of user credentials/contact info.
- A third-party email marketing tool (e.g., Mailchimp, SendGrid, HubSpot) is used for promotional emails.
- Password reset links are generated using the email stored in the auth service.
- There may be a cache layer (e.g., Redis) storing user profile data for performance.

---

## Test Ideas

### Data Consistency Validation
- After updating email, immediately trigger a password reset — verify which email address receives the reset link.
- Check the value stored in the primary user DB, auth service DB, and marketing service — all three should reflect the new email.
- Test with a deliberate delay (e.g., 1 min, 1 hour, 24 hours) between email update and triggering a reset — check if sync happens eventually.
- Query each database/service directly via API or DB tool to confirm the new email is stored.

### Cache Invalidation Testing
- After email update, check if the API still returns the old email for a short period (cache hit).
- Force a cache flash and retest — does the new email appear immediately?
- Simulate a scenario where the cache TTL is very long (e.g., 24 hours) and verify that email communications use the cached (old) email during that window.

### Third-Party Email Service Sync Testing
- Check if the system has a sync job that updates contacts in the email marketing platform — test its frequency and reliability.
- Simulate an email update and verify the contact is updated in the marketing tool (e.g., via Mailchimp API audit).
- Test what happens if the sync job fails — is there a retry? An alert? A fallback?
- Verify that unsubscribe/re-subscribe status is preserved when the email is updated.

### Auth Service Sync Testing
- Confirm that the email update event is published to the auth service (via event bus or API call).
- Test what happens if the auth service is temporarily down when the email update occurs — is the update retried later?
- Test the password reset flow end-to-end after an email change.

---

## Edge Cases

- User changes email multiple times in quick succession — which email does the auth service end up with?
- Email update event is published but the auth service consumer is down — event is lost if the queue is not durable.
- Old email is deleted from the mail provider but the system still tries to send to it — bounces silently.
- User changes email to one that already exists in the system (another account) — conflict handling.
- Marketing email is triggered from a scheduled batch job that reads from a snapshot/backup table not updated in real time.
- User opts out of marketing emails, then changes email — new email is re-enrolled without consent.
- Cache invalidation only happens for the session that made the change — other sessions still have stale cache.

---

## Risks

- **Security risk:** Password reset links going to old email — potentially accessible by someone else who now owns that address.
- **Compliance risk:** Sending promotional emails to an address the user no longer owns may violate GDPR or CAN-SPAM regulations.
- **User trust risk:** User sees correct email in UI but receives communications on old email — causes confusion and distrust.
- **Data quality risk:** Fragmented email data across systems leads to unreliable communication and analytics.
- **Support overhead:** Users contact support about receiving emails on old addresses — manual correction required.
- **Deliverability risk:** If old email is no longer valid, email bounces damage sender reputation and deliverability scores.
