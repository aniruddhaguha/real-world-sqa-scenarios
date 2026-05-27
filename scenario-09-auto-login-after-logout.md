# Scenario 09: User Auto-Logged In After Logout and App Restart

## Scenario

A user logs out of the mobile app and closes it completely. When **reopening the app later**, they are **automatically logged in again** without entering credentials.


---

## Assumptions

- The app uses token-based authentication (e.g., JWT access token + refresh token).
- Logout is supposed to clear all local session data and invalidate the server-side session.
- Tokens may be stored in SharedPreferences (Android), UserDefaults (iOS), Keychain, or secure storage.
- The app may use biometric/auto-login features that persist credentials separately.
- The issue may be related to incomplete token clearing on logout.

---

## Test Ideas

### Token & Session Storage Testing
- After logout, inspect local storage (SharedPreferences/UserDefaults/Keychain) — verify all tokens are cleared.
- After logout, inspect the app's local database and cache — confirm no session-related data remains.
- After logout and app restart, intercept network calls — check if the app sends any auth token in headers.
- Test if the logout API call invalidates the refresh token on the server side.

### Server-Side Session Invalidation Testing
- After logout, manually use the old access token to call a protected API — it should return 401 Unauthorized.
- After logout, manually use the old refresh token to request a new access token — it should be rejected.
- Verify the server maintains a token blacklist or invalidated session store.
- Test what happens if the logout API call fails (network error) — does the app still clear local tokens?

### Auto-Login & Biometric Testing
- Check if the app has a "Remember Me" or biometric login feature — and whether logout properly disables it.
- Test if the operating system's credential manager (Android Smart Lock, iOS iCloud Keychain) is restoring credentials.
- Verify that logout explicitly removes any saved credentials from the OS-level credential store.
- Test on devices with multiple user accounts — confirm session isolation.

### App Lifecycle Testing
- Test the exact flow: Login → Use App → Logout → Force Close → Reopen — reproduce the auto-login.
- Test with different methods of closing: swiping from recent apps vs. force stop vs. device restart.
- Test after an OS update — check if the update restores previously cleared data from a backup.
- Check if the app restores state from a crash recovery mechanism that includes auth tokens.

---

## Edge Cases

- OS-level app backup (Android Auto Backup, iOS iCloud backup) restores tokens after app reinstall or device restore.
- Refresh token has a very long expiry (e.g., 30 days) and is not cleared on logout — auto-refresh happens on next open.
- The app uses a `WebView` with persistent cookies — cookies survive logout and auto-authenticate.
- Multiple sessions: user is logged in on two devices — logging out on one does not affect the other, which may auto-resume.
- Token stored in a location not covered by the logout clearing routine (e.g., stored in a file, not in SharedPreferences).
- Deep link opens the app post-logout and silently re-authenticates using a cached token in the URL handler.
- App uses SSO (Single Sign-On) — SSO session persists even after app-level logout.

---

## Risks

- **Security risk (Critical):** A shared or lost device could give unauthorized access to another person's account.
- **Privacy risk:** Sensitive user data (messages, transactions, personal info) is accessible without re-authentication.
- **Compliance risk:** Apps handling financial or health data must ensure session termination on logout — failure may violate GDPR, PCI-DSS, or HIPAA.
- **Account takeover risk:** If a device is stolen post-logout, the attacker gains full account access.
- **Trust risk:** Users who intentionally log out expect their session to end — auto-login violates this expectation.
- **Multi-user device risk:** On shared devices (family phones, work phones), auto-login exposes one user's account to another.
