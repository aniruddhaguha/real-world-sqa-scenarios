# Scenario 15: Referral Bonus Exploited Multiple Times for the Same Person

## Scenario

A referral program rewards users for inviting friends. Some users manage to receive **referral bonuses multiple times for the same person** by slightly changing emails or timing registrations.

---

## Assumptions

- The referral program grants a bonus when a referred user registers and completes a qualifying action (e.g., first purchase).
- Referral uniqueness is validated by the referred user's email address.
- The system does not validate other uniqueness factors (device, IP, phone number, payment method).
- Bonuses are credited automatically upon qualifying event.
- The attacker may create slightly different email variations (e.g., `user+1@gmail.com`, `user+2@gmail.com`) that point to the same inbox.

---

## Test Ideas

### Email Normalization Testing
- Test registration with Gmail's `+` alias trick: `user@gmail.com` and `user+ref@gmail.com` — verify they are treated as the same email.
- Test case sensitivity: `User@gmail.com` vs. `user@gmail.com` — verify normalized to the same.
- Test with dots in Gmail: `us.er@gmail.com` vs. `user@gmail.com` — Gmail treats these as the same; verify the system does too.
- Test with Googlemail vs. Gmail: `user@googlemail.com` vs. `user@gmail.com`.
- Test with disposable email domains (e.g., Mailinator, Guerrilla Mail) — verify if they are blocked.

### Duplicate Detection Testing
- Test registering the same user twice with slightly different email variations — verify only one referral bonus is granted.
- Test timing attacks: register User B using Referral A, earn bonus, deactivate User B, re-register with a different email under Referral A.
- Test if phone number verification prevents the same person from registering multiple times.
- Test if device fingerprinting or IP-based limits prevent self-referral loops.

### Self-Referral Testing
- Test if User A can refer themselves using a different email.
- Test circular referrals: A refers B, B refers A — both should not earn unlimited bonuses.
- Test if a referrer can earn from their own new account (e.g., old account refers new account of same person).

### Timing & Race Condition Testing
- Test two simultaneous referral registrations from the same device — does the system deduplicate?
- Test if a referral completion event fires before duplicate detection logic runs — bonus may be granted before the duplicate check.
- Test if bonuses can be triggered multiple times by replaying the qualifying event (e.g., completing a purchase multiple times on different test accounts).

---

## Edge Cases

- User creates accounts using multiple phone numbers (SIM cards) — system relies only on email for uniqueness.
- User uses a VPN to change IP between registrations — IP-based detection is bypassed.
- User waits for the referral bonus to credit, deletes the account, and registers again with the same referral link.
- Referring user creates 100 fake "friend" accounts using email aliases — earns 100 bonuses.
- Referral link is posted publicly — mass exploitation by unintended users.
- Referral bonus is tied to first purchase — attacker makes a minimal purchase, earns bonus, requests a refund.
- New user completes the qualifying action but the referral bonus is granted to a different referrer (link manipulation).

---

## Risks

- **Financial risk:** Fraudulent referral payouts represent direct monetary loss to the business.
- **Program viability risk:** Abuse can drain the referral program budget entirely, forcing its cancellation.
- **Legal risk:** Fraudulent use of promotional programs may violate terms of service — enforcement requires detection.
- **Competitive distortion risk:** Legitimate users are disadvantaged when fraudsters accumulate disproportionate bonuses.
- **Data quality risk:** Fake user accounts created for referral abuse pollute analytics and user databases.
- **Reputation risk:** If the abuse becomes public, it may signal weak fraud controls to investors and partners.
