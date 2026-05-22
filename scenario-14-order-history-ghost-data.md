# Scenario 14: Order History Disappears but Reappears After Refresh

## Scenario

Users report that their **order history sometimes disappears completely**, but **reappears after refreshing** or logging out and back in.

---

## Assumptions

- Order history is fetched from a backend API and displayed in the app.
- There is a caching layer (client-side or server-side) that stores recent order history responses.
- Multiple backend instances or read replicas may serve the order history API.
- The UI may use a local data store (Redux, SQLite, local cache) to display orders between API calls.
- The issue is intermittent — not reproducible on every request.

---

## Test Ideas

### Caching Layer Testing
- Force-clear the app cache and observe if order history loads correctly from the API.
- Check if the client caches the order history response — test what happens when the cache is stale or corrupted.
- Test the cache invalidation mechanism: after placing a new order, does the cache update or serve stale data?
- Simulate a cache miss scenario — verify the app fetches from the API and displays correctly.

### API Inconsistency Testing
- Hit the order history API multiple times in quick succession — check if responses are consistent.
- If the backend uses read replicas, test if replication lag causes empty results on some reads.
- Verify API pagination: if the page is loaded incorrectly (e.g., wrong page number or empty page), order history could appear blank.
- Test the API response when there is a temporary database connection issue — does it return empty array `[]` or an error?

### State Management Testing
- Inspect the local state (Redux store, MobX, ViewModel) during the disappearance — is the order list actually empty or just not rendered?
- Test if a navigation action or lifecycle event (e.g., app going to background) clears the local state incorrectly.
- Verify that the order history component fetches fresh data on mount vs. relying on cached state.
- Test with React DevTools or platform-equivalent — monitor state changes when order history disappears.

### Session & Auth Testing
- Check if a session expiry or silent token refresh causes the API to briefly return unauthorized data.
- Test if order history is scoped to the correct user ID — ensure the API uses the authenticated user, not a cached user reference.
- Verify that logging out and back in forces a fresh API call (which is why the refresh "fixes" it).

---

## Edge Cases

- Read replica is slightly behind the primary — a new order appears on primary but not on replica; the user lands on the replica.
- CDN or reverse proxy caches a 200 OK empty response and serves it to all users for the cache duration.
- Pagination offset is misapplied — if user is on page 2, a state reset takes them to page 1 with wrong offset, returning empty results.
- App goes to background, OS reclaims memory, state is cleared — on resume, the UI shows empty state before the API call completes.
- Network request races: old request returns after a new one, overwriting the populated list with an empty or stale list.
- Token refresh happens mid-request — the original order history request uses the old token, which is rejected, returning empty.
- Soft delete issue: orders are being marked as "deleted" incorrectly and then restored — appears as disappear/reappear.

---

## Risks

- **User trust risk:** Disappearing data is alarming — users may believe their order history is actually deleted.
- **Support overhead risk:** Users contacting support about missing orders create unnecessary workload.
- **Data integrity perception risk:** Even if data is intact, intermittent display issues undermine confidence in the platform.
- **Conversion risk:** If order history disappears during the return/refund process, it may block legitimate customer actions.
- **Debugging difficulty risk:** Intermittent, non-reproducible issues are among the hardest to diagnose and fix.
- **Caching strategy risk:** Over-aggressive caching for performance optimization can cause data visibility issues at scale.
