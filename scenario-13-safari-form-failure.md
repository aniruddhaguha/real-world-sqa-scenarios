# Scenario 13: Form Submission Works in Chrome/Edge but Fails Silently in Safari

## Scenario

A form submission **works perfectly in Chrome and Edge** but **fails silently in Safari** — no error shown, no data stored.

---

## Assumptions

- The form uses standard HTML or a JavaScript framework (React, Vue, Angular).
- The form data is submitted via a JavaScript fetch/XHR call (not a native HTML form POST).
- Safari uses WebKit engine, which has different behavior for certain JavaScript APIs, CSS, and network policies.
- "Fails silently" means Safari does not display an error in the UI, and no data reaches the server.
- The issue may be related to Safari's stricter privacy policies, ITP (Intelligent Tracking Prevention), or missing polyfills.


---

## Test Ideas

### Cross-Browser Debugging
- Open Safari's Web Inspector (Develop menu) and monitor the Console and Network tabs during form submission.
- Compare network requests in Chrome DevTools vs. Safari Web Inspector — identify any request that is missing or different in Safari.
- Check the Console for JavaScript errors specific to Safari (e.g., unsupported API usage, strict mode errors).
- Replay the form submission using `curl` with the exact headers Safari sends — test if the server processes it.

### JavaScript Compatibility Testing
- Check if any JavaScript used in the form relies on APIs not supported in Safari (e.g., newer `fetch` options, `AbortController`, `FormData` edge cases).
- Test if the form uses `type="submit"` correctly or relies on a custom `onClick` handler that Safari handles differently.
- Verify that Promises and async/await are polyfilled if targeting older Safari versions.
- Check for `SameSite` cookie issues — Safari blocks cross-site cookies, which may affect authentication during form submission.

### Network & Security Testing
- Check if the form POSTs to a different origin (CORS) — Safari may block the request if CORS headers are missing or incorrect.
- Verify CORS headers on the server: `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`.
- Test if Safari's ITP is blocking cookies or referrer headers that the server requires for authentication.
- Check if the form submission uses `credentials: 'include'` in fetch — and if the server correctly handles preflight OPTIONS requests.

### Frontend Difference Testing
- Check CSS and layout differences — confirm form fields are visible and interactable in Safari.
- Test form input types (e.g., `type="date"`, `type="color"`) — Safari renders some differently and may prevent submission.
- Verify that form validation (HTML5 `required`, `pattern`) behaves consistently — Safari may block submission differently.
- Test on both macOS Safari and iOS Safari — they have slightly different WebKit implementations.

---

## Edge Cases

- Safari blocks the request due to a missing or incorrect `Content-Type` header (e.g., `application/json` vs. `multipart/form-data`).
- Safari's strict referrer policy strips the `Referer` header — if the server uses this for CSRF validation, the request is silently rejected.
- Form submission is blocked by Safari's built-in popup blocker (if it opens a new window on submit).
- `localStorage` or `sessionStorage` used for token storage is blocked in Safari's private browsing mode — auth token missing from request.
- Date input fields send differently formatted values in Safari — server validation fails silently.
- Safari enforces a stricter Mixed Content policy — HTTPS page submitting to HTTP endpoint is blocked.
- Safari auto-fill populates fields in a way the JavaScript validation doesn't expect — form appears complete but fails the check.


---

## Risks

- **User exclusion risk:** Safari is the default browser on all Apple devices — failing silently locks out a significant portion of users.
- **Revenue risk:** If the failing form is a checkout, registration, or lead capture, silent failure directly impacts conversions.
- **Debugging difficulty risk:** Silent failures with no console error are extremely hard to diagnose without a Mac/iPhone test device.
- **iOS-specific risk:** iOS mandates use of WebKit for all browsers — even Chrome on iOS uses WebKit, so the issue may affect all iOS users.
- **Security risk:** If CORS is loosened to fix the issue, it may introduce vulnerabilities for other browsers.
- **Reputation risk:** Apple device users experiencing silent failures may perceive the product as low-quality.
