# Scenario 10: Currency Switch Updates Listing Prices but Checkout Remains in Old Currency

## Scenario

An e-commerce platform allows users to change currency (USD, EUR, BDT). After switching currency, **product listing prices update correctly**, but **checkout total remains in the old currency** — causing users to pay wrong amounts.

---

## Assumptions

- Currency preference is stored in the user's session or profile.
- Product listing prices are calculated on the frontend using the selected currency and exchange rates.
- The checkout/cart service is a separate backend service that calculates total price independently.
- The payment gateway charges in a specific currency sent in the checkout payload.
- Exchange rates may be fetched from a third-party API and cached.

---

## Test Ideas

### Currency State Propagation Testing
- Switch currency from USD to EUR, add a product to cart — verify the cart API request includes the correct currency code.
- Check the checkout API request payload — confirm `currency` field reflects the user's current selection.
- Switch currency multiple times quickly — verify the final selected currency is what gets sent to checkout.
- Test currency switch after adding items to cart vs. before — confirm cart totals update in both cases.

### Frontend–Backend Integration Testing
- Use a proxy (Charles, Fiddler) to inspect what currency is sent in the checkout API call.
- Compare the currency in the product listing response vs. the cart/checkout response — they should match.
- Verify the cart service reads currency preference from the session/user profile, not from a hardcoded default.
- Test the checkout page total display — is it re-calculated on load using current currency, or is it a cached value?

### Payment Gateway Integration Testing
- Confirm the currency code sent to the payment gateway matches the displayed checkout currency.
- Test a complete purchase flow in each supported currency (USD, EUR, BDT) — verify correct currency appears on payment confirmation and receipt.
- Verify that exchange rate conversion (if applied at checkout) uses the current rate, not a stale cached rate.
- Test what happens if the payment gateway does not support the selected currency — does it silently fall back?

### Session & Cookie Testing
- Check if currency preference is stored in a cookie, local storage, session, or user profile.
- Test currency persistence across page refreshes, new tabs, and sessions.
- Test in incognito mode — confirm the currency preference behavior for guest users.

---

## Edge Cases

- User switches currency while the checkout page is already open — total does not update without a page refresh.
- Currency is set to EUR but the payment gateway defaults to USD — user pays in USD without realizing.
- Exchange rate is stale (cached for 24 hours) — price shown on listing differs slightly from checkout total.
- Guest user changes currency — preference stored in a cookie — but the server-side cart session uses the default currency.
- User has EUR selected, adds items to cart, currency switches to BDT (automatically on geo-location change), then proceeds to checkout — which currency is used?
- User on mobile app changes currency — changes not reflected in active web session (and vice versa).
- Promotional discount is applied in old currency and not recalculated after currency switch.
---

## Risks

- **Financial risk:** Users pay the wrong amount — either overpaying or underpaying — leading to disputes and refunds.
- **Legal/compliance risk:** Charging a different amount than what was displayed may violate consumer protection laws.
- **User trust risk:** Currency mismatch between listing and checkout is a significant UX failure that erodes trust.
- **Revenue risk:** Users who discover the mismatch at checkout may abandon their purchase.
- **International market risk:** Multi-currency support is critical for global platforms — inconsistency limits market viability.
- **Fraud risk:** If checkout consistently charges in a weaker currency, sophisticated users may exploit this intentionally.

