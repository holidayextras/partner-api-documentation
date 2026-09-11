# Migrating airport parking from V1 to Partner API 2026

This guide is for partners who currently sell airport parking through the Holiday Extras V1 API. It explains what changes in Partner API 2026, how the journeys map, and what to consider as you move across.

Partner API 2026 uses V2 endpoints and contracts. It is a new contract rather than a like-for-like change of endpoint paths, so you will need to update your authentication, request and response models, price handling, retry behaviour, and post-booking journey.

Your Holiday Extras partnerships contact will work with you on access, testing, and launch timing. To get connected with the right person, email [partnerconnect@holidayextras.com](mailto:partnerconnect@holidayextras.com).

---

## At a glance

| Area | What changes in Partner API 2026 |
|---|---|
| Authentication | OAuth 2.0 client credentials replace request-level agent details and V1 user tokens |
| Search and content | Structured JSON responses replace V1 route, market, and `fields` variations |
| Product selection | The selected result includes a `product_token` carrying the product, journey, and price context needed to book |
| Price protection | Each eligible result includes a Price Lock deadline, with price validation at booking |
| Safe retries | Booking and confirmation requests require an `Idempotency-Key` |
| Manage booking | Retrieval, amendment quote/confirm, and cancellation quote/confirm use dedicated endpoints |
| Booking updates | A `booking.impacted` webhook tells you when a booking has changed, in place of polling |
| Parking access | Ordered `access_methods` replace constructed barcode and QR-code URLs |
| Errors | HTTP status codes and RFC 9457 Problem Details replace V1 result and error structures |

---

## Before you start

### Confirm your current scope

Make a short inventory of the V1 features your integration uses. Include:

- countries, currencies, airports, and parking products;
- availability by airport or by individual car park;
- Product Library fields and any V1 `fields` merged into availability;
- booking request flags, customer details, vehicle details, and flight details;
- price checking and price-change handling;
- amendments, cancellations, refunds, and customer service processes;
- barcodes, QR codes, confirmations, and pre-travel communications;
- any supplements, upgrades, or other partner-specific behaviour.

Partner API 2026 currently supports airport parking. Support for port parking, supplements, and upgrades will follow. If your V1 journey uses any of these, your Holiday Extras contact will help you agree the right migration timing.

### Request Partner API 2026 access

Partner API 2026 authenticates with OAuth 2.0 `client_credentials`, not the request-level agent details and user tokens V1 used. Holiday Extras will provide a `client_id` and securely share a `client_secret` for each environment as you move through onboarding. You exchange those for a short-lived bearer access token at `POST https://auth.holidayextras.com/oauth2/token` and send it in the `Authorization` header on every request.

Keep `client_secret` and access tokens securely on your server and out of browsers, mobile apps, URLs, and logs.

Start with the [Partner API 2026 onboarding guide](../../onboarding.md), the [authentication guide](../integration-guides/02-authentication.md), and the [token endpoint reference](../api-reference/post-auth-token.md).

### Existing V1 bookings

Bookings created through V1 can be retrieved, amended, and cancelled through the Partner API 2026 parking endpoints using their Holiday Extras booking reference. This gives you one Partner API 2026 manage-booking journey for both existing V1 bookings and new bookings.

---

## Endpoint mapping

Each Partner API 2026 endpoint below links to its full reference page, covering parameters, request and response shapes, and error responses.

| Journey step | V1 | Partner API 2026 (V2) | What to change |
|---|---|---|---|
| Get credentials or token | V1 user-token flow; send agent details, `key`, and `token` with requests | [`POST https://auth.holidayextras.com/oauth2/token`](../api-reference/post-auth-token.md) | Use OAuth `client_credentials`, cache the access token, and refresh it before expiry |
| List parking locations | `GET /v1/location?type=1` for UK or `type=carpark&system=de` for Europe | [`GET /v2/locations?product_types=parking`](../api-reference/get-locations.md) | Read `supported_codes`, countries, product types, and currencies from the response |
| Search an airport | `GET /v1/carpark/{AirportCode}` | [`GET /v2/products/parking/detailed`](../api-reference/get-parking-availability-detailed.md) or [`GET /v2/products/parking`](../api-reference/get-parking-availability.md) | Pass `location_type=iata`, `location_code`, currency, and either parking or flight datetimes |
| Search one car park | `GET /v1/carpark/{CarParkCode}` | [Search the location](../api-reference/get-parking-availability.md), then select the matching result `code` | Use the returned `product_token` in place of a product-specific booking URL |
| Get content | `GET /v1/product/{productCode}` or availability `fields` | [Detailed search](../api-reference/get-parking-availability-detailed.md), or [`GET /v2/content/parking?product_codes=...`](../api-reference/get-parking-content.md) | Use `accept-language`; cache by product code and language; allow for nullable fields |
| Check price | `GET /v1/carpark/{productCode}/priceCheck`, or price check during booking | [Search](../api-reference/get-parking-availability.md) returns the price and [Price Lock](../integration-guides/07-price-lock.md); booking validates the price automatically | Use the selected result's `pricing`, `product_token`, `price_lock_valid_until`, and `product_token_valid_until` |
| Create booking | `POST /v1/carpark/{productCode}` | [`POST /v2/bookings/parking`](../api-reference/post-parking-booking.md) | Send JSON with the `product_token`, customer, declared product requirements, partner reference, and a unique `Idempotency-Key` |
| View booking | `GET /v1/booking/{ref}` | [`GET /v2/bookings/parking/{ref}`](../api-reference/get-parking-booking.md) | Retrieve a V1- or V2-created booking using its Holiday Extras booking reference |
| Amend booking | GET/POST `/v1/booking/{ref}`, depending on amend type | [`PATCH .../amendments/quote`](../api-reference/patch-parking-amendment-quote.md), then [`POST .../amendments/confirm`](../api-reference/post-parking-amendment-confirm.md) | Quote the change, show any price outcome, then confirm with the returned token and an idempotency key |
| Cancel booking | GET/POST `/v1/booking/{ref}` with `ConfirmCancel=N` then `Y` | [`GET .../cancellations/quote`](../api-reference/get-parking-cancellation-quote.md), then [`POST .../cancellations/confirm`](../api-reference/post-parking-cancellation-confirm.md) | Show the refund from the quote, then confirm with the cancellation token and an idempotency key |
| Get entry method | Booking response plus V1 barcode or QR-code URL | [`access_methods` from `GET /v2/bookings/parking/{ref}`](../api-reference/get-parking-booking.md) | Follow the returned priority order and handle values that become available after supplier fulfilment |
| Receive later updates | Poll `GET /v1/booking/{ref}` | Signed [`booking.impacted` webhook](../integration-guides/05-webhooks.md), followed by `GET` booking | Retrieve the latest booking when the webhook arrives |

---

## Request and field mapping

### Authentication and request format

| V1 | Partner API 2026 |
|---|---|
| `ABTANumber`, `Password`, `key`, V1 `token`, and sometimes `Initials` on API requests | Bearer access token in the `Authorization` header |
| XML or `application/x-www-form-urlencoded` booking body | `application/json` booking body |
| `.js` URL suffix for a JSON response | JSON is the standard response format |
| `System=ABC` or `System=ABG` | Replaced by one standard Partner API 2026 contract across supported markets |
| `lang` query parameter | `accept-language` header on endpoints that return localised content |

### Search

| V1 field or behaviour | Partner API 2026 equivalent |
|---|---|
| Airport code in the path | `location_type=iata` and `location_code` query parameters |
| `ArrivalDate` + `ArrivalTime` | `parking_entry_datetime` |
| `DepartDate` + `DepartTime` | `parking_exit_datetime` |
| `OutFlight` | `outbound_flight_number`; you can also send the outbound scheduled datetime |
| Return-flight details | `inbound_flight_number` and `inbound_arrival_datetime` |
| `System` parameter to select a market | Not required. One contract covers every supported location |
| Currency implied by the chosen system | `currency` query parameter, set per search. Each location's `supported_currencies` lists what it accepts |
| `fields` added to availability | Use detailed search, or merge separate search and content responses by product code |
| `RequestFlags` or `CarDetFlags` | Replaced by the `product_requirements` declared by the selected result |
| `TotalPrice` | `pricing.total.amount_major`, `amount_minor`, and `currency` |
| `BookingURL` | `product_token`; create all parking bookings through `POST /v2/bookings/parking` |
| `CanAmendCantCancel` or advance-purchase content | Structured `policies.amendments` and `policies.refunds` |

Partner API 2026 accepts either a parking window or the relevant flight datetimes. If you search by flight times, the API derives the parking entry and exit times. In every case, use the confirmed `parking_entry_datetime` and `parking_exit_datetime` returned on the selected product.

Local datetimes use the airport's local time and omit a timezone offset. Timestamps ending in `Z`, including token, Price Lock, policy, and requirement deadlines, are UTC.

### Booking

| V1 field | Partner API 2026 field or behaviour |
|---|---|
| Product code in the booking path | Included in the search result's `product_token` |
| `ArrivalDate`, `ArrivalTime`, `DepartDate`, `DepartTime` | Included in the `product_token`, so you can omit them from the booking body |
| `PriceCheckFlag` and `PriceCheckPrice` | Price Lock and product-token validation at booking |
| `CustomerRef` | `partner_reference` |
| `Title` | Use the customer's given and family names; these provide the details needed to create and manage a parking booking |
| `Initial` | `customer.given_name` accepts the customer's first name |
| `Surname` | `customer.family_name` |
| `Email` | `customer.email` |
| Address, town, county, and postcode | Omit these from the current parking booking request |
| `Registration` | `product_requirements.vehicle_registration` |
| `CarMake` | `product_requirements.vehicle_make` |
| `CarModel` | `product_requirements.vehicle_model` |
| `CarColour` | `product_requirements.vehicle_colour` |
| `OutFlight` or `OutFltNo` | `product_requirements.outbound_flight_number` |
| `ReturnFlight` or `InFltNo` | `product_requirements.inbound_flight_number` |
| `OutTerminal` request flag | `product_requirements.outbound_terminal_code`, when declared by the selected product |
| `ReturnTerminal` request flag | `product_requirements.inbound_terminal_code`, when declared by the selected product |
| `MobileNum` | `product_requirements.mobile_number` |
| `Destination` | `product_requirements.destination` |
| `NumberOfPax` | Not needed for search. If declared by the selected product, send it as `product_requirements.number_of_passengers` |

Read `product_requirements` from the selected search result rather than maintaining a fixed list by product. Use `required_at` to understand whether each detail is needed at booking or before travel.

### Current scope notes

Partner API 2026 currently focuses on airport parking. Support for port parking, supplements, and upgrades will follow.

| V1 field or capability | Migration action |
|---|---|
| `ShipName` and `PierName` | Planned as part of future port parking support |
| `ChildSeat` and `AddlServices` | Outside the current airport parking scope; your Holiday Extras contact can help you plan around these fields |
| Supplements and upgrades | Planned for a future release |

Send the fields returned by the selected Partner API 2026 result and accepted by the relevant endpoint. Your Holiday Extras contact will help you agree the migration timing for any remaining V1 fields that are important to your products or customer journey.

---

## Ready to migrate

- [ ] Every V1 field and capability you use has a confirmed Partner API 2026 outcome.
- [ ] Search and booking stay within the same version for each new-sale journey.
- [ ] Existing V1 bookings can be retrieved and, where their policies allow, amended and cancelled through Partner API 2026.
- [ ] Your journeys use the returned price, `product_token`, Price Lock, policies, and `product_requirements`.
- [ ] Booking and confirmation retries reuse the idempotency key for the same customer action.
- [ ] Confirmation and pre-travel journeys use the latest fulfilment and `access_methods` details.
- [ ] Webhooks prompt your systems to retrieve the latest booking, if you use them.
- [ ] The full journey works in staging across your agreed products, markets, currencies, and languages.
- [ ] Production credentials, launch timing, contacts, and escalation routes are agreed.

---

## Partner API 2026 reference

- [API overview](../integration-guides/01-api-overview.md)
- [Authentication](../integration-guides/02-authentication.md)
- [Search endpoints](../integration-guides/04-search-endpoints.md)
- [Selling parking](../user-guides/selling-parking.md)
- [Price Lock](../integration-guides/07-price-lock.md)
- [Accessing parking](../user-guides/accessing-parking.md)
- [Webhooks](../integration-guides/05-webhooks.md)
- [Error handling](../errors.md)
- [Sandbox testing](../integration-guides/03-sandbox-testing.md)
- [Create a parking booking](../api-reference/post-parking-booking.md)
- [View a parking booking](../api-reference/get-parking-booking.md)
- [Quote an amendment](../api-reference/patch-parking-amendment-quote.md)
- [Quote a cancellation](../api-reference/get-parking-cancellation-quote.md)
