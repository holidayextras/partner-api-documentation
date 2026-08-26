# Price Lock

Price Lock makes it easier to carry a selected price from search through to booking with confidence.

Holiday Extras protects the quoted `pricing.total` against eligible price increases until `price_lock_valid_until`. This keeps the usual checkout journey simple: you can show the price, collect the customer's details and create the booking without checking the price again at every step.

That means less integration complexity for you, fewer interruptions during checkout and a smoother experience that can help support conversion.

---

## How Price Lock works

Every parking result includes `price_lock_valid_until`, a UTC datetime showing when its Price Lock window ends. The quoted `pricing.total` is protected against eligible price increases until that time.

Use the timestamp returned for the selected product in your checkout logic. The booking endpoint checks the latest price and availability before creating the booking.

---

## Where Price Lock is available

Every parking result returned by either parking search endpoint includes `price_lock_valid_until`:

- `GET /v2/products/parking`
- `GET /v2/products/parking/detailed`

We'll update this guide as Price Lock becomes available for more products.

---

## Price Lock and product-token validity

`price_lock_valid_until` and `product_token_valid_until` represent two separate times.

| Field | Meaning | How to use it |
|---|---|---|
| `price_lock_valid_until` | The quoted `pricing.total` is protected against eligible price increases until this time | Use it to understand whether the selected price remains locked |
| `product_token_valid_until` | The product token can be submitted to attempt a booking until this time | Use it to understand whether the selected product can still be submitted to booking |

A product token can remain valid after its Price Lock window has ended. You can still attempt to create the booking while the token is valid. The API checks the latest price and availability before creating the booking.

The longer a product token is held before booking, the greater the chance that the price or availability may have changed. This varies by product, supplier and time of year, including periods of higher or lower demand.

---

## Continuing through checkout

| Situation | How to continue |
|---|---|
| The Price Lock and product token are valid | Continue checkout with the selected token and price |
| The Price Lock has ended, but the product token is valid | You can attempt the booking. Be ready to handle a price change or the product becoming unavailable |
| The product token has expired | Run a fresh search |
| Booking returns `409 Conflict` with `validated_price` and `alternative_product_token` | Choose whether to show the updated price or absorb the difference |
| Booking returns `409 Conflict` without both replacement fields | Run a fresh search |
| The selected product is no longer available | Run a fresh search and help the customer choose another option |

---

## If the price has changed

Price changes at booking that are not absorbed by Price Lock are expected to be exceptional. They are most likely when the booking is attempted after the Price Lock window or the supplier price has increased significantly. Product availability may also have changed since the search.

When the API returns `409 Conflict` with both `validated_price` and `alternative_product_token`, the product is available to book at the updated API price. You can choose the approach that best fits your commercial model:

1. **Show the updated price.** Explain the change and ask the customer to accept the new price before continuing with the alternative product token.
2. **Absorb the difference.** Keep the customer's original price and use the alternative product token to book at the updated API price, with your business covering the difference.

If the response does not include both fields, run a fresh search before presenting another bookable price.

---

## Related documentation

- [API overview](./01-api-overview.md)
- [Search endpoints](./04-search-endpoints.md)
- [Displaying prices](../user-guides/displaying-prices.md)
- [Search parking](../api-reference/get-parking-availability.md)
- [Search parking with content](../api-reference/get-parking-availability-detailed.md)
- [Create a parking booking](../api-reference/post-parking-booking.md)
- [Error handling](../errors.md)
- [Sandbox testing](./03-sandbox-testing.md)
- [Sandbox scenario catalogue](./08-sandbox-scenarios.md#price-lock-and-price-validation)

---

Next: [Webhooks](./05-webhooks.md)
