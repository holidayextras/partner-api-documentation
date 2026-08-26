# Selling parking as a standalone product

Use this path when customers are mainly here to find and compare airport parking. They may still have wider travel plans, but parking is the product they are actively choosing in this journey.

The API supports a fuller comparison experience: broad product lists, richer product detail, filters, sorting, and post-booking self-service.

---

## What to focus on

| Focus | Why it helps | Useful docs |
|---|---|---|
| Show enough choice to compare | Customers expect to weigh up service type, price, distance, transfer time, and cancellation flexibility | [Selling parking](../selling-parking.md) |
| Use clear filters and sorting | Longer product lists are easier to use when customers can narrow the options | [Search parking with content](../../api-reference/get-parking-availability-detailed.md) |
| Put richer content on detail pages | Address, directions, procedures, images, facilities, and restrictions help customers choose with confidence | [Parking content endpoint](../../api-reference/get-parking-content.md) |
| Group close variants where useful | Similar products may differ by cancellation flexibility, EV charging, or vehicle size | [Grouping products with `site_name`](../selling-parking.md#grouping-products-with-site_name) |
| Make post-booking details easy to revisit | Customers often come back later for directions, access details, changes, or cancellations | [Accessing parking](../accessing-parking.md) |

---

## Suggested shape

Start with a search form built around airport, entry date and time, exit date and time, language, and currency. In the results, keep product cards scannable and use detail pages for the fuller explanation.

Useful filters usually include service type, price, distance, transfer time, cancellation flexibility, EV charging, accessibility, and vehicle size where those fields are available.

For product cards, prioritise the fields that help customers compare quickly: product name, parking type, price, cancellation terms, transfer time or walking distance, and one clear selling point.

For detail pages, focus on the practical information that helps customers choose: address, distance, facilities, security details, restrictions, and product requirements. Arrival and departure procedures can sit in the detail view where they explain how a service works, but they are often more useful in confirmation and pre-travel reminders.

---

## Integration notes

1. Cache `GET /v2/locations` daily for supported airports and currencies.
2. Use `GET /v2/products/parking/detailed` when search results need content and pricing together.
3. Use `site_name` to group variants of the same car park where that improves comparison.
4. Show cancellation terms, product requirements, and vehicle restrictions before selection.
5. Book with `POST /v2/bookings/parking`.
6. Build manage-booking around `GET /v2/bookings/parking/{ref}`, amendment quote/confirm, and cancellation quote/confirm where you want self-service.
7. Use webhooks if your own dashboard or reminder emails need to stay aligned with supplier updates.
