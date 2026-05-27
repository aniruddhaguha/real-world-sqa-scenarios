# Scenario 07: Money Transfer Shows Success but Receiver Gets Credited Twice

## Scenario

A banking app processes money transfers and shows **"Transfer Successful"** instantly. Later, some users complain that the **receiver received the same amount twice**, while their own balance was deducted only once.

---

## Assumptions

- The app uses a REST or async API to initiate transfers.
- The payment/transfer service communicates with a core banking system or ledger.
- The transfer involves a debit from sender and a credit to receiver — ideally in one atomic transaction.
- A retry mechanism exists (either client-side or server-side) to handle network failures.
- The system may use a message queue to process transfer events asynchronously.
- There is no idempotency key being enforced, or it is implemented incorrectly.


---

## Test Ideas

### Idempotency Testing
- Send the same transfer request twice with the same payload — verify only one transaction is processed.
- Test with an `Idempotency-Key` header (if supported) — send duplicate requests with the same key and confirm deduplication.
- Remove the idempotency key and resend — verify if duplicates occur.
- Test the server's behavior when it receives a request it has already processed but hasn't returned a response for yet.

### Retry Mechanism Testing
- Simulate a network timeout after the server processes the transfer but before the response is sent — observe if the client retries.
- Verify that client-side retry logic uses idempotency keys.
- Test server-side retry: if the core banking system fails mid-credit, does the system retry the credit only (not the debit again)?
- Simulate a message queue retry: if the credit event is not acknowledged, does the queue redeliver it — and does the receiver get credited again?

### Rollback & Failure Recovery Testing
- Simulate a failure after debit succeeds but before credit is processed — what is the system state?
- Verify that rollback correctly reverses both debit and credit in failure scenarios.
- Test compensating transactions: if the credit fails, does the system issue a debit reversal?
- Test what happens if the rollback itself fails — is there a dead-letter queue or manual intervention process?

### Transaction Atomicity Testing
- Confirm that debit and credit are wrapped in a distributed transaction (e.g., 2-phase commit or saga pattern).
- Test with a deliberate failure injected between debit and credit — verify atomicity holds.
- Test concurrent transfers from the same account — confirm balance is correctly managed.

---

## Edge Cases

- Transfer request sent while the app is offline — queued locally and resent when connectivity returns, potentially multiple times.
- Core banking system processes the credit but fails to acknowledge — middleware resends and double-credits.
- Same transfer triggered from two devices simultaneously (e.g., user taps the button on mobile while a scheduled job also fires).
- Transfer confirmation webhook delivered twice by the payment gateway — system processes both.
- Clock skew between systems causes a "duplicate" request to appear as a new one (idempotency window expired).
- Receiver's account is in a different bank — credit goes through an intermediary that has its own retry logic.
- System upgrade/migration during a live transaction results in the transaction being processed by both the old and new system.

---

## Risks

- **Financial risk:** Double credit means the bank/platform loses money equal to the duplicate amount.
- **Regulatory risk:** Incorrect account credits may violate central bank regulations and audit requirements.
- **Reconciliation risk:** Ledger imbalances are difficult to detect and fix without robust audit trails.
- **Legal risk:** Users may refuse to return duplicate credits, citing the bank's error.
- **Trust risk:** If users discover inconsistencies, it severely damages confidence in the banking app.
- **Detection risk:** Double credits may not be caught until end-of-day reconciliation — giving time for funds to be withdrawn.
