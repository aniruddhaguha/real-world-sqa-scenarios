# SQA Roadmap Scenarios

This directory contains a collection of 20 real-world Software Quality Assurance (SQA) scenarios. These scenarios represent common, complex edge cases and bugs found in modern applications, providing a comprehensive resource for practicing testing strategies, debugging, system analysis, and quality assurance.

## Scenarios Index

- [Scenario 01: Payment Confirmed but Order Never Received by Restaurant](./scenario-01-payment-order-sync.md)
- [Scenario 02: Login Works on Wi-Fi but Fails on Mobile Data](./scenario-02-login-wifi-vs-mobile-data.md)
- [Scenario 03: Flash Sale Overselling — Out of Stock After Payment Confirmed](./scenario-03-flash-sale-overselling.md)
- [Scenario 04: Email Update Shows in UI but Old Email Receives System Emails](./scenario-04-email-update-sync.md)
- [Scenario 05: Ride Starts Automatically Without Driver Action](./scenario-05-ride-auto-start.md)
- [Scenario 06: Videos Show 100% Complete but Certificate Remains Locked](./scenario-06-certificate-not-unlocking.md)
- [Scenario 07: Money Transfer Shows Success but Receiver Gets Credited Twice](./scenario-07-duplicate-credit-transfer.md)
- [Scenario 08: Push Notifications Arrive Hours Late or Not at All](./scenario-08-push-notification-delays.md)
- [Scenario 09: User Auto-Logged In After Logout and App Restart](./scenario-09-auto-login-after-logout.md)
- [Scenario 10: Currency Switch Updates Listing Prices but Checkout Remains in Old Currency](./scenario-10-currency-checkout-mismatch.md)
- [Scenario 11: Two Users Book the Same Seat Simultaneously](./scenario-11-double-seat-booking.md)
- [Scenario 12: Large Image Upload Freezes App with No Error Message](./scenario-12-large-image-upload-freeze.md)
- [Scenario 13: Form Submission Works in Chrome/Edge but Fails Silently in Safari](./scenario-13-safari-form-failure.md)
- [Scenario 14: Order History Disappears but Reappears After Refresh](./scenario-14-order-history-ghost-data.md)
- [Scenario 15: Referral Bonus Exploited Multiple Times for the Same Person](./scenario-15-referral-bonus-fraud.md)
- [Scenario 16: Long-Time Users Crash at Checkout After Major App Update](./scenario-16-checkout-crash-old-users.md)
- [Scenario 17: Analytics Dashboard Values Jump Backward Instead of Increasing](./scenario-17-analytics-backward-jump.md)
- [Scenario 18: Chat Messages Show "Delivered" but Receiver Never Gets Them](./scenario-18-message-delivered-but-not-received.md)
- [Scenario 19: Subscription Cancelled but Premium Access Remains Active for Weeks](./scenario-19-subscription-access-after-cancel.md)
- [Scenario 20: Old Password Works for Several Minutes After Password Change](./scenario-20-old-password-valid-after-change.md)

## Scenario Structure

Each markdown file follows a structured approach to analyzing the issue, which includes:
1. **Scenario**: A clear description of the problem.
2. **Assumptions**: The system architecture context and underlying technical conditions.
3. **Test Ideas**: Detailed testing approaches covering Flow Tracing, API Validation, Log & DB Validation, and UI Validation.
4. **Edge Cases**: Unusual conditions that could lead to or exacerbate the issue.
5. **Risks**: The business, financial, and technical impacts of the bug.
