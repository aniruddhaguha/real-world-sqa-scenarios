# Scenario 20: Old Password Works for Several Minutes After Password Change

## Scenario

After changing a password successfully, the user can still **log in using the old password for several minutes** — a serious security vulnerability.

---

## Assumptions

- Password change updates the user's credential in the primary authentication database.
- There is a cache layer (Redis, Memcached) storing user credentials or session data for performance.
- Active sessions may use JWTs with long expiry periods, not invalidated on password change.
- The authentication service reads credentials from the cache, not directly from the primary database, for performance.
- Existing sessions may remain valid until the cached data expires or the token expires.

---

## Test Ideas

### Cache Invalidation Testing
- Change password and immediately try logging in with the old password — note how long it remains valid.
- Query the cache directly after password change — verify the old credential hash is invalidated.
- Check the cache TTL for user credential records — this TTL defines the window of vulnerability.
- Test if the password change API explicitly calls the cache to delete/update the credential entry.
- Verify cache invalidation works across all nodes in a distributed cache cluster.

### Session & Token Invalidation Testing
- After changing password, attempt to use existing active sessions (browser tabs, other devices) — they should be invalidated.
- Verify that all active JWT tokens are invalidated after a password change (e.g., via token version/jti blacklist).
- Test: Log in on Device A and Device B. Change password on Device A. Verify Device B is logged out.
- Check if the system increments a `password_version` or `token_version` field and validates it on each request.

### Authentication Service Testing
- Test the authentication service's read path: does it check the cache first, then the database? Verify cache-aside pattern is implemented correctly.
- Test a write-through cache: when the password is updated in the database, is the cache updated simultaneously?
- Test a write-around cache: when the password is updated, is the cache invalidated immediately or allowed to expire?
- Simulate a cache invalidation failure — verify the system falls back to the database.

### Security Validation Testing
- Test the password change flow with a compromised session: change password, use old session token — should fail immediately.
- Verify CSRF protection on the password change endpoint.
- Test if the system requires re-authentication (current password entry) before allowing a password change.
- Test if the password change event generates a security alert/email to the user.
- Verify the password reset flow (forgot password) also invalidates all existing sessions.

---

## Edge Cases

- Distributed cache has multiple nodes — invalidation is sent to one node but not propagated to others immediately (eventual consistency).
- Old password login succeeds but the session created with it is silently rejected on the next request (confusing UX).
- User changes password, then changes it back to the old one — old password becomes valid again; previous sessions may or may not be invalidated.
- API gateway caches the authentication result (user is authenticated) for a short period — old credentials pass through cached auth.
- Concurrent requests: user changes password and simultaneously logs in with old password — race condition determines which wins.
- Mobile app stores credentials locally and retries silently — auto-login with old password succeeds due to cache.
- Password change via admin panel does not trigger the same cache invalidation as a user-initiated change.

---

## Risks

- **Security risk (Critical):** The window of old-password validity allows an attacker who obtained the old password to continue accessing the account even after the owner changes it.
- **Account takeover risk:** In an active attack scenario (compromised credentials), the attacker can maintain access despite the user's mitigation attempt.
- **Compliance risk:** PCI-DSS, OWASP, and ISO 27001 require immediate invalidation of credentials upon change — this vulnerability is a direct compliance failure.
- **Trust risk:** Users who discover their old password still works after changing it will lose all confidence in the platform's security.
- **Incident response risk:** If this is discovered during a breach response, the cache window may give attackers continued access while the team believes they've secured the account.
- **Regulatory risk:** Depending on the industry (banking, healthcare), this vulnerability may trigger mandatory breach notifications and regulatory penalties.
