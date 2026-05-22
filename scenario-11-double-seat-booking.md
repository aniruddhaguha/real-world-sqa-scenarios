# Scenario 11: Two Users Book the Same Seat Simultaneously

## Scenario

A movie ticket booking system allows **two users to reserve the same seat at the exact same time** during peak hours. Both users receive confirmation messages, but only one can enter the hall.

---

## Assumptions

- Seat availability is stored in a relational database.
- Booking flow: User selects seat → seat is "held" temporarily → user pays → booking is confirmed.
- The system runs multiple application server instances behind a load balancer.
- The temporary hold mechanism may use a timer-based lock (e.g., 5-minute hold).
- Peak hours create high concurrency — many users attempting to book the same popular seats simultaneously.
- No distributed locking is currently applied, or it is applied incorrectly.

---

## Test Ideas

### Concurrency Control Testing
- Write a load test that sends two simultaneous booking requests for the same seat — verify only one succeeds.
- Test with N concurrent requests (N = 10, 50, 100) for the same seat — confirm total confirmed bookings equal 1.
- Use JMeter or k6 to simulate peak-hour concurrency and observe database behavior.
- Check database transaction isolation level — `READ COMMITTED` may allow phantoms; `SERIALIZABLE` prevents them but reduces performance.

### Database Locking Testing
- Verify that seat selection uses `SELECT ... FOR UPDATE` to lock the row during booking.
- Test optimistic locking: if two users try to book simultaneously, the second should receive a conflict error (e.g., "seat already taken").
- Check if the seat status is updated atomically — read and write should happen in a single transaction.
- Simulate a delayed transaction (hold seat but slow payment) — confirm seat is released correctly on expiry.

### Real-Time Availability Testing
- Check how seat availability is communicated to users in real time.
- Test if the seat map refreshes automatically when another user books a seat.
- Verify that "seat held" status is visible to other users browsing the same screen.
- Test what happens if a held seat expires — does it become available immediately for others?

### API & State Validation Testing
- Confirm booking API has idempotency handling — duplicate booking requests should not create duplicate records.
- Test the booking confirmation flow: payment success → confirm booking → update seat status — verify this is atomic.
- Test seat status consistency between the booking service and the display service after confirmation.

---

## Edge Cases

- Two users on different application server instances select the same seat — each server reads "available" before either writes.
- User A holds a seat, payment times out — seat is released. User B's booking was queued — it gets processed after release and succeeds. User A retries and also succeeds.
- Cache shows seat as available but database shows it as booked (stale cache).
- Same user double-taps the "Book" button — two requests sent simultaneously — seat booked twice on two tickets.
- Third-party ticketing aggregator sends a booking request at the same time as a direct booking.
- Seat is held by User A, User A's session expires — seat released — User B books — User A's original payment completes and tries to confirm booking.
- Network timeout causes the booking response to never reach User A — user retries and gets a duplicate booking.

---

## Risks

- **Customer experience risk:** One confirmed user cannot enter the hall — leads to strong complaints and potential legal action.
- **Revenue risk:** Issuing refunds for double-booked seats has operational cost.
- **Reputation risk:** Double booking is a classic system failure that significantly damages brand image, especially for popular events.
- **Scalability risk:** Implementing proper locking (e.g., `SELECT FOR UPDATE`) can reduce throughput — must be balanced carefully.
- **Cascade failure risk:** Under extreme peak load, database deadlocks may occur if locking is not implemented carefully.
- **Legal risk:** In some jurisdictions, a confirmed booking constitutes a contract — overbooking may have legal consequences.
