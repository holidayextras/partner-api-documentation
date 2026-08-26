# Selling parking alongside a flight

Use this path when parking is offered in a flight-led journey, such as flight booking, manage booking, check-in, pre-travel emails, or an app itinerary view.

Parking works best when it uses the flight context the customer already understands: airport, terminal, dates, times, and travel reminders.

---

## What to focus on

| Focus | Why it helps | Useful docs |
|---|---|---|
| Use flight context | Flight number, airport, terminal, dates, and times help keep parking relevant | [API overview](../../integration-guides/01-api-overview.md) |
| Keep the choice simple | The customer is usually completing another journey, so a compact parking choice works well | [Selling parking](../selling-parking.md) |
| Use high-intent moments | Booking, manage booking, check-in, and reminders are natural places to offer parking | [Customer journey](../customer-journey.md) |
| Keep access details close to travel | Directions, arrival procedure, and access method matter most when the customer is thinking about the airport | [Accessing parking](../accessing-parking.md) |
| Plan for booking changes | Flight changes may create parking amendment or cancellation needs | [Amendment quote](../../api-reference/patch-parking-amendment-quote.md) |

---

## Suggested shape

Offer parking when the customer is already thinking about the airport: after flight selection, in manage booking, during check-in, and in pre-travel reminders. Use the flight information you already hold to pre-fill the search and reduce customer effort.

If you search using flight times, send the outbound departure from the airport where the customer will park and the inbound arrival back at that same airport. Including `outbound_flight_number` helps return terminal-relevant options at larger airports.

For in-path journeys, show a small set of relevant options with clear differences. The most useful fields are service type, terminal or distance, transfer time, cancellation flexibility, and total price.

---

## Integration notes

1. Map flight itinerary data to parking search inputs.
2. Use `outbound_flight_number` where available to improve terminal relevance.
3. Search with `GET /v2/products/parking/detailed` when the journey needs product content and price together.
4. Book with `POST /v2/bookings/parking` and connect the Holiday Extras booking reference to the flight or booking record.
5. Include the hosted confirmation page link in pre-travel emails, manage-booking pages, or app journeys.
6. Use reminder emails or app notifications before travel to surface the latest parking details.
7. Decide whether amendments and cancellations are self-serve or handled by customer service.
