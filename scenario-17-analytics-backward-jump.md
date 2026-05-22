# Scenario 17: Analytics Dashboard Values Jump Backward Instead of Increasing

## Scenario

A real-time analytics dashboard updates live numbers, but occasionally the **values jump backward** instead of increasing.

---

## Assumptions

- The dashboard consumes a real-time data stream (e.g., Kafka, WebSocket, SSE).
- Metrics are cumulative (e.g., total sales, total signups) — they should only increase.
- Events may arrive out of order due to network latency or distributed processing.
- The dashboard may be calculating totals by summing incoming event values, not reading a single authoritative source.
- Multiple data producers (microservices, regions) are writing to the same stream.

---

## Test Ideas

### Timestamp Handling Testing
- Inject events with out-of-order timestamps into the stream — observe how the dashboard processes them.
- Verify that the dashboard uses event time (the time the event occurred) not processing time (the time it was received).
- Test with events delayed by 5 seconds, 30 seconds, 5 minutes — confirm late arrivals are handled correctly (e.g., using watermarks).
- Confirm the dashboard has a defined strategy for late events: discard, update, or apply to the correct window.

### Data Streaming Order Testing
- Produce a sequence of events in a known order and verify they are consumed in the same order.
- Test with a multi-partition Kafka topic — ordering is only guaranteed within a partition; verify cross-partition ordering is handled.
- Simulate a producer restart — check if it resends old events and how the consumer handles duplicates.
- Test the consumer's offset management — if offset is committed incorrectly, old events may be replayed.

### Sync Delay Testing
- Check dashboard refresh rate vs. data stream latency — if the dashboard polls too frequently, it may show partial aggregations.
- Test with a slow or intermittent stream — observe dashboard behavior when data arrives in bursts.
- Verify that partial windows (e.g., an incomplete minute) are clearly labeled or excluded from the display.
- Simulate a 30-second stream pause followed by a burst of events — observe how the dashboard handles the catch-up.

### Aggregation Logic Testing
- Verify that the aggregation (sum, count) is idempotent — reprocessing the same event should not change the result.
- Test if the dashboard recalculates from scratch or applies incremental updates — recalculation should produce a consistent result.
- Test with event values that include corrections (negative values) — verify they are applied correctly without causing the total to drop unexpectedly.
- Simulate a time zone change or DST transition — check if event bucketing is affected.

---

## Edge Cases

- Two producers write events with the same timestamp from different time zones — treated as duplicates or processed incorrectly.
- Consumer group rebalancing in Kafka causes some events to be reprocessed — totals are temporarily inflated then corrected.
- Dashboard applies a sliding window that drops old events — total drops as events age out of the window.
- Clock drift between producer and consumer servers causes events to appear from the "past."
- Event schema version changes mid-stream — old and new events are parsed differently, leading to incorrect values.
- High-frequency micro-updates (many events per second) arrive faster than the dashboard can aggregate — race condition in the aggregation state.
- Negative correction event (e.g., refund, cancellation) processed before the original event — causes a temporary backward jump.

---

## Risks

- **Decision-making risk:** Incorrect real-time metrics lead to wrong business decisions (e.g., triggering a sale prematurely, misreading demand).
- **Trust risk:** Business stakeholders who see metrics jumping backward lose confidence in the analytics system.
- **Alert risk:** Monitoring thresholds based on these metrics may fire false positives or negatives.
- **Audit risk:** Backward jumps in financial metrics (revenue, transactions) raise red flags during audits.
- **Debugging difficulty risk:** Out-of-order events in distributed systems are notoriously hard to trace and reproduce.
- **SLA risk:** If the dashboard is used for real-time operational decisions, incorrect data could breach SLAs.
