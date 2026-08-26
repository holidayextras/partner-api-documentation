# Customer journey - airport parking

Before you go live, map the full parking journey in your own customer experience. A good integration does more than take payment - it helps customers choose the right product, arrive at the right place, and know what to do if their plans change.

Use this checklist as the minimum customer journey to support:

| Journey stage | What your customer needs | Partner API guidance |
|---|---|---|
| Search | Parking shown at the right point in the trip flow, using the airport, dates, times, currency, and flight details you already know | Include `outbound_flight_number` where available so results can be filtered to the right terminal |
| Product choice | A clear reason to choose one product over another | Show service type, price, cancellation flexibility, transfer time or walking distance, security signals, and the product selling point |
| Checkout | Only the details needed to complete the booking | Use `product_requirements` to collect fields required at booking. For `"before_travel"` requirements, such as vehicle registration, you can collect them after purchase and submit them by amendment |
| Confirmation | A booking reference and a reliable place to find the latest parking details | After booking, call `GET /v2/bookings/parking/{ref}` and include `confirmation_page_link` in your confirmation email or account area |
| Before travel | Practical arrival instructions, access details, and any missing details the car park still needs | Refetch the booking before sending reminders. Anchor follow-ups for missing details on each item's `requirement_deadline` in `product_requirements` |
| At the car park | The right address, arrival procedure, and way to enter the car park | Use the access guidance from the view-booking response. If the supplier sends instructions directly, make that clear to the customer |
| Changes and cancellations | A clear quote before the customer commits to a change or cancellation | Use the quote-then-confirm amendment and cancellation flows, and show the customer any price change or refund before confirming |
| Booking details change after checkout | Confidence that late supplier updates still reach them when they matter | Use webhooks as a prompt to refetch the booking. Contact the customer only if the latest details affect their trip, such as a new access method, supplier reference, barcode, or arrival instruction |

If you search by flight times instead of direct parking times, send the outbound flight departure from the airport where the customer will park, and the return flight arrival back at that same airport. The API uses those flight times to work out the parking entry and exit window, so the customer has time to park before departure and collect their car after landing.

Different partner journeys can use the same API pattern:

- **Holiday or flight booking partners** should keep parking in the booking path and make the default choice easy to scan.
- **Parking comparison and aggregator partners** should help customers compare a fuller set of products with filters, sorting, and richer detail pages.
- **Airlines and tour operators** should connect parking to the trip the customer already understands, then use reminder emails or trip dashboards to keep access details close to the flight.
- **Technology providers and implementation partners** should use the journey pattern that matches the travel brand they are integrating for.

For guidance on presenting products before booking, see [Selling parking](./selling-parking.md). For the post-booking part of the journey, see [Accessing parking](./accessing-parking.md).
