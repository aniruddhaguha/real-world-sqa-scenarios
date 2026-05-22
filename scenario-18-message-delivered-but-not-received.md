# Scenario 18: Chat Messages Show "Delivered" but Receiver Never Gets Them

## Scenario

A chat app shows messages as **"Delivered,"** but the receiver **never actually gets them.**

---

## Assumptions

- "Delivered" status is shown based on an acknowledgment from the server — not confirmation from the recipient's device.
- The server updates message status to "Delivered" when it accepts the message, not when the recipient's device receives it.
- Message delivery to the recipient uses WebSocket, FCM/APNs push, or polling.
- The recipient may be offline, and messages should be queued and delivered when they come online.
- The issue is systemic — not just one recipient, but a broader group.

---

## Test Ideas

### Message Acknowledgment Flow Testing
- Map the full delivery lifecycle: Sender → Server (Sent) → Server acknowledges (Delivered?) → Recipient device (Received) → Recipient opens (Read).
- Verify exactly what event triggers the "Delivered" status — is it server receipt or recipient device receipt?
- Implement a test where the recipient device is offline — send a message and confirm "Delivered" is NOT shown until the device comes online and receives it.
- Compare "Delivered" with true delivery by checking if the message appears in the recipient's chat history.

### Push Notification & Real-Time Testing
- Send a message to a recipient with push notifications disabled — verify the message is queued and delivered when the app is opened.
- Send a message via WebSocket while the recipient is connected — verify real-time delivery.
- Kill the WebSocket connection mid-message — verify the message is not lost and is delivered on reconnect.
- Test FCM/APNs delivery of message notifications separately from WebSocket delivery.

### Queue & Storage Testing
- Verify that undelivered messages are stored in a persistent queue or database.
- Test what happens when the queue is full or the storage limit is reached — are oldest messages dropped?
- Check message TTL (time to live) — if the recipient is offline for days, are messages eventually dropped?
- Verify messages are marked as delivered only after the queue confirms the recipient has received them.

### "Delivered" vs. "Received" Semantic Testing
- Clearly define what "Delivered" means in the system: server received? Device received? App received? User notified?
- Test a scenario where the server stores the message but the recipient's device never pulls it — what status is shown?
- Test with a recipient who has uninstalled the app — message is undeliverable; what status shows on sender's side?
- Verify the status flow for group messages — "Delivered" should only show when all recipients have received.

---

## Edge Cases

- Recipient has blocked the sender — message is stored on server but never delivered; sender sees "Delivered."
- Recipient's device token (FCM/APNs) is stale — push notification fails silently; server marks as delivered.
- Message is delivered to the recipient's device but the app is force-stopped — notification shown but message not in-app.
- Server sends message to wrong device (recipient has multiple devices) — delivered to one, not shown on the device they're using.
- Message encryption failure: message delivered but recipient's app cannot decrypt it — shows as garbled or not at all.
- Network partition: sender's server accepts the message but cannot route to the recipient's server (different nodes in a distributed system).
- Message delivered to an old session/device that the recipient no longer uses.

---

## Risks

- **Trust risk:** "Delivered" status is a core reliability signal — false delivery statuses break the fundamental promise of the chat app.
- **User experience risk:** Recipients missing messages may miss time-sensitive communications (work instructions, emergencies).
- **Legal/compliance risk:** In business messaging (e.g., healthcare, legal), non-delivery of confirmed messages can have serious consequences.
- **Debugging risk:** The mismatch between UI status and actual delivery may go unnoticed for extended periods.
- **Retention risk:** Users who cannot rely on message delivery will switch to competing chat applications.
- **False confidence risk:** Senders believe their messages were received and act accordingly — leading to miscommunication.
