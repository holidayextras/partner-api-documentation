# Parking integration paths

The Partner API shape stays the same, but the right airport parking journey depends on where parking appears in your booking flow.

Use these guides to choose the path that best matches your customer journey, then follow the linked integration and user guides for the detailed build.

| Journey context | Best fit | Start here |
|---|---|---|
| Standalone parking | Parking is the main product, and customers expect to compare options | [Selling parking as a standalone product](./standalone-parking.md) |
| Wider travel booking | Parking is offered alongside a holiday, flight, hotel, ferry, package, or wider travel checkout | [Selling parking as part of a wider travel booking](./travel-booking.md) |
| Flight-led journey | Parking needs to feel connected to the flight, terminal, and travel timeline | [Selling parking alongside a flight](./flight-led.md) |

---

## What stays the same

All paths use the same core API pattern:

1. Authenticate with OAuth 2.0.
2. Discover supported locations and currencies.
3. Search for parking using the travel details you already hold.
4. Create a booking using the selected `product_token`.
5. Fetch the booking for confirmation and access details.
6. Manage amendments and cancellations through quote-then-confirm flows.
7. Use webhooks where your post-booking journey needs to stay in sync.

For the full technical flow, see [API overview](../../integration-guides/01-api-overview.md).

---

## How to choose

If parking is the destination product, start with the standalone parking path. Customers will expect richer comparison tools, filters, product detail pages, and confidence-building content.

If parking is an ancillary in a wider travel checkout, start with the wider travel booking path. Keep the choice simple and let customers add parking without slowing down the main booking.

If the customer already has a flight, start with the flight-led path. Use the airport, terminal, dates, and reminder moments that already exist in the flight journey.

If you provide the backend or middleware for another brand, use the path that matches the customer journey they are building. The technical patterns stay the same: authentication, search, booking, caching, webhooks, error handling, and support diagnostics are covered in the integration guides.

---

## Useful shared guides

- [Customer journey](../customer-journey.md)
- [Selling parking](../selling-parking.md)
- [Accessing parking](../accessing-parking.md)
- [Displaying prices](../displaying-prices.md)
- [Sandbox testing](../../integration-guides/03-sandbox-testing.md)
- [Error handling](../../errors.md)
