# Scenario 02: Login Works on Wi-Fi but Fails on Mobile Data

## Scenario

After releasing a new mobile app version, users report they can log in successfully when connected to **Wi-Fi** but fail when using **mobile data**. The backend team says the authentication service is working fine and sees no errors in normal logs.

---

## Assumptions

- The app uses HTTPS for all API calls.
- Authentication involves token-based auth (e.g., JWT or OAuth2).
- The mobile app was recently updated — backend was not changed.
- "Fail" means the login attempt either returns an error, times out, or silently does nothing.
- Some users experience the issue; it is not 100% reproducible for all users on mobile data.
- Backend logs show no failures because requests may not be reaching the server at all.

---

## Test Ideas

### Network Configuration Testing
- Test login on different mobile carriers (Grameenphone, Robi, Banglalink, etc.) to isolate carrier-specific issues.
- Use a Wi-Fi hotspot from a mobile device to see if the issue persists (to differentiate between mobile data protocols vs. carrier firewalls).
- Test with a VPN enabled on mobile data — if login works with VPN, the issue is likely carrier-level firewall or DNS filtering.
- Check if the app's base URL or API endpoint is accessible on mobile data using a browser.

### Timeout & Retry Testing
- Simulate high-latency networks using throttling tools (e.g., Charles Proxy, Network Link Conditioner).
- Test with packet loss simulation to see if login fails without proper retry logic.
- Check if the app has different timeout values for mobile vs. Wi-Fi connections.
- Verify if the app retries failed login attempts — and if so, with correct exponential backoff.

### API Security Rules Testing
- Check if the backend uses IP whitelisting or rate-limiting that blocks mobile data IP ranges.
- Verify if SSL certificate pinning in the new app version rejects certificates on mobile data (possible if mobile carrier uses transparent proxies).
- Test if any WAF (Web Application Firewall) or CDN rule blocks requests from mobile IPs.
- Inspect request headers on mobile data vs. Wi-Fi — check if `User-Agent`, `X-Forwarded-For`, or other headers differ.

### Client-Side Logic Testing
- Capture network traffic on mobile data using a proxy (Charles, Proxyman) — check if requests are actually sent.
- Check if the app uses a different API endpoint or configuration for mobile data.
- Verify if any network-type check in the code (`isWifi()`) incorrectly blocks API calls.
- Test with the previous app version on mobile data — confirm if regression was introduced in this release.

---

## Edge Cases

- Mobile carrier acts as a transparent proxy and breaks SSL — causing certificate mismatch errors.
- App uses IPv6 on mobile data but the server only supports IPv4.
- Mobile data has a different MTU (Maximum Transmission Unit), causing large packets (e.g., with tokens) to be fragmented and fail.
- Background data restrictions on Android/iOS prevent the app from making API calls on mobile data.
- DNS resolution fails on mobile data for the app's domain — returns a different IP.
- Mobile data connection drops mid-authentication request — app does not handle partial response gracefully.
- New app version has a bug where it checks for Wi-Fi before allowing login (accidental feature flag or misplaced condition).

---

## Risks

- **User access risk:** Users on mobile data (often the majority in mobile-first markets) are locked out.
- **Revenue risk:** Failed logins lead to abandoned sessions, reduced engagement, and potential churn.
- **Carrier-specific discrimination:** If only certain carriers are affected, it may appear as a niche bug but affect a large user segment.
- **Security risk:** SSL pinning misconfiguration could expose users on mobile data to MITM attacks.
- **Detection risk:** Backend shows no errors because requests never reach the server — passive monitoring will miss this entirely.
- **Reputation risk:** Users may report the app as broken on app stores, hurting ratings.
