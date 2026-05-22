# Scenario 01: Payment Confirmed but Order Never Received by Restaurant

## Scenario

A food delivery app confirms payment and shows the order as **"Placed Successfully."** However, the restaurant dashboard never receives the order, while the customer's money is already deducted. Customer support can see the payment record but no order record in the admin panel.

---

## Assumptions

- The system uses separate microservices for payment and order management.
- Payment gateway sends a webhook/callback to the backend after successful payment.
- Order creation is triggered after payment confirmation — not simultaneously.
- There is a message queue or event bus (e.g., Kafka, RabbitMQ) between payment and order services.
- Logs exist for each service independently.
- The UI receives a success response from the payment service, not the order service.

---

## Test Ideas

### Flow Tracing
- Trace the full request lifecycle: UI → Payment Gateway → Payment Service → Order Service → Restaurant Dashboard.
- Validate that the payment callback/webhook is reaching the backend correctly.
- Verify that the event or message published after payment is being consumed by the order service.
- Check if the order service successfully writes to its database after consuming the event.
- Validate that the restaurant dashboard is polling or subscribing to the correct order event/topic.

### API Validation
- Test the payment confirmation API response — does it return `order_id` along with `payment_id`?
- Test the order creation API directly (bypass payment) — does it create an order in the restaurant dashboard?
- Simulate a delayed/failed webhook from the payment gateway and observe how the system responds.
- Check idempotency: if the webhook is delivered twice, is only one order created?

### Log & Database Validation
- Check payment service logs for successful transaction records.
- Check order service logs for any errors during order creation (e.g., DB write failure, timeout).
- Compare records in payment DB vs. order DB — find the gap.
- Check the message queue for dead-letter queues or unprocessed messages.
- Verify admin panel reads from order DB (not a cached or separate source).

### UI Validation
- Check what triggers the "Placed Successfully" message — is it based on payment success or order creation confirmation?
- Reproduce the scenario and capture network calls from the UI.

---

## Edge Cases

- Payment gateway sends webhook but it times out before the order service processes it.
- Order service is down at the moment the event is published — message is lost if the queue is not durable.
- Database transaction for order creation fails silently (no rollback, no retry).
- Race condition: payment confirmed but order service hasn't started processing yet — UI shows success prematurely.
- Restaurant dashboard subscribes to a wrong topic/channel.
- Order service receives the event but fails to notify the restaurant due to a separate push/notification failure.
- Network partition between payment service and order service during the transaction.

---

## Risks

- **Financial risk:** Customer is charged but receives no service — leads to refund requests and loss of trust.
- **Data inconsistency:** Payment and order databases are out of sync — difficult to reconcile at scale.
- **Silent failure:** No error alerts fired, making it hard to detect the issue proactively.
- **Customer support bottleneck:** Support sees payment but cannot find order — manual investigation required.
- **Scalability risk:** Under high load, the message queue or order service may drop events more frequently.
- **Regulatory/compliance risk:** Charging customers without delivering service may violate payment regulations.
