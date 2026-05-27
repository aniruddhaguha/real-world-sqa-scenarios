# Scenario 03: Flash Sale Overselling — Out of Stock After Payment Confirmed

## Scenario

During a massive flash sale, thousands of users add the same limited-stock product to their carts. Some users manage to complete payment, but later receive **cancellation messages** saying "Out of stock," while others who paid **later** successfully receive the product.

---

## Assumptions

- The product has a limited, fixed inventory count stored in a database.
- Inventory is decremented at the time of order confirmation or payment.
- Multiple application servers are running behind a load balancer.
- The system does not use distributed locking or pessimistic locking for inventory.
- Cart reservation may or may not hold inventory temporarily.
- The payment gateway is a third party; payment confirmation arrives asynchronously.

---

## Test Ideas

### Concurrency & Race Condition Testing
- Write load tests (e.g., using k6, JMeter, Locust) that simulate thousands of simultaneous "Place Order" requests for a product with stock of 10.
- Verify that the total confirmed orders never exceed the available stock.
- Test with multiple application server instances — simulate what happens when two servers read stock = 1 simultaneously and both confirm orders.
- Use database query logs to check if inventory reads and writes happen atomically.

### Inventory Locking Testing
- Test if the system uses `SELECT ... FOR UPDATE` (pessimistic locking) or optimistic locking (version/ETag) when decrementing inventory.
- Simulate a scenario where two requests read inventory at the same time before either writes — confirm if overselling occurs.
- Test that a failed payment properly releases any reserved inventory.
- Verify inventory count after each load test run matches expected values.

### Transaction Atomicity Testing
- Confirm that payment and inventory decrement happen in a single atomic transaction — or are properly compensated if one fails.
- Test rollback: simulate a payment success followed by an inventory write failure — does the system roll back, cancel the order, or leave inconsistent state?
- Test the behavior when the database goes under high load — does it drop transactions or process them incorrectly?

### Cart Reservation Testing
- Test if adding to cart temporarily reserves inventory, and for how long.
- Simulate a user holding items in cart without completing payment — does this block others from purchasing?
- Test what happens when cart reservation expires while payment is being processed.


---

## Edge Cases

- Two users click "Pay" at the exact same millisecond for the last available unit.
- Payment gateway callback arrives out of order — a later payment is confirmed before an earlier one.
- One server decrements inventory to 0 while another server's cached value still shows stock > 0.
- A user refreshes the checkout page during payment — triggering duplicate payment attempts.
- Network timeout causes a user's payment to be retried — resulting in two orders from one user.
- Inventory is stored in a cache (Redis) that is not synchronized with the database.
- A background job restores cancelled order stock but does so incorrectly (adds too much or too little).

---

## Risks

- **Overselling risk:** Customers pay for products that cannot be delivered — severe trust and refund cost.
- **Unfair experience risk:** Users who paid earlier receive cancellations while later-payers succeed — damages brand reputation.
- **Financial risk:** Processing refunds at scale has operational cost and payment gateway fees.
- **Scalability risk:** Locking mechanisms that prevent race conditions may create bottlenecks under peak load.
- **Legal risk:** In some markets, confirming an order creates a legally binding contract — cancelling it may have legal implications.
- **Data integrity risk:** Inventory count may become negative or mismatched between services if not properly managed.
