# Scenario 19: Subscription Cancelled but Premium Access Remains Active for Weeks

## Scenario

A user **cancels a subscription**, gets a **cancellation confirmation email**, but still has **premium access for weeks** after the expected end date.

---

## Assumptions

- Subscription status is managed by a billing service (Stripe, Paddle, or custom).
- Access control (feature permissions) is managed by a separate entitlement or permission service.
- Cancellation marks the subscription as "cancelled at period end" — user should retain access until the end of the current billing period, then lose it.
- A scheduled job (cron job) or webhook should trigger access revocation at the correct time.
- "Weeks" of extra access suggests the revocation mechanism failed entirely, not just a minor delay.

---

## Test Ideas

### Cancellation Flow Testing
- Test end-to-end: cancel a subscription with a known billing period end date — verify premium access is removed exactly at that date.
- Test immediate cancellation (cancel and revoke now) vs. end-of-period cancellation — verify both behaviors work correctly.
- Verify the cancellation confirmation email mentions the correct access end date.
- Test cancellation via all available methods: in-app, web, customer support — verify each correctly updates the system.

### Billing System & Webhook Testing
- Verify the billing provider sends a webhook on subscription end (e.g., `customer.subscription.deleted` in Stripe).
- Test what happens when the webhook fails to deliver — does the system retry? Is there a fallback?
- Simulate a webhook delivery failure and verify the subscription access is still revoked (polling-based fallback).
- Check if the webhook handler correctly processes the payload and updates the entitlement service.

### Entitlement & Permission Sync Testing
- Verify that the entitlement service is notified immediately when a subscription is cancelled or expires.
- Check if access control is read from a cache — and if the cache is invalidated upon cancellation.
- Test reading the user's entitlement after cancellation — confirm it reflects "free tier" or "no access."
- Test if access is re-evaluated on each request (real-time) or periodically (batch) — batch evaluation leads to delays.

### Cron Job / Scheduled Task Testing
- Identify the scheduled job responsible for revoking access — test it manually for a cancelled subscription.
- Verify the cron job frequency and confirm it runs reliably (check job logs, failure alerts).
- Test what happens when the cron job fails or skips a run — does access remain active until the next successful run?
- Verify that the cron job handles time zones correctly — ensure it revokes access at midnight of the correct time zone.

---

## Edge Cases

- Billing period ends on a weekend or holiday — cron job runs but the entitlement service is under maintenance.
- User cancels just before the billing cycle renews — billing service processes the renewal before the cancellation, extending access by one more cycle.
- Subscription is cancelled but a pending invoice succeeds — billing service treats the subscription as renewed.
- Entitlement service caches the "premium" status for 24 hours — user retains access for up to 24 hours after revocation.
- Webhook is delivered and processed, but a database transaction failure means the status was never actually updated.
- User re-subscribes immediately after cancelling — confuses the system into treating the original cancellation as invalid.
- Access is controlled by a JWT with a long expiry — even after cancellation, the existing token grants access until it expires.

---

## Risks

- **Revenue loss risk:** Users with free premium access do not need to renew — direct financial impact.
- **Fairness risk:** Paying users subsidize non-paying users who retain access — undermines the business model.
- **Audit risk:** If premium access is not correctly revoked, revenue reporting and active subscriber counts are inaccurate.
- **Legal risk:** If the cancellation confirmation email specifies an access end date that is not honored correctly, the company may be in breach of its own terms.
- **Detection risk:** Extended free access is rarely complained about by users — the issue may go unnoticed for months.
- **Compliance risk:** In regulated industries (e.g., financial data, healthcare), continued access after cancellation may violate data access agreements.
