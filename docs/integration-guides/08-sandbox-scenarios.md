# Sandbox scenario catalogue

Use this catalogue when you need a consistent API response for a specific scenario.

Sandbox uses fake data. Each documented scenario returns a predictable response, allowing you to inspect it and check how your integration handles it. Repeating the same documented request returns the same scenario.

Use staging to test the complete end-to-end journey with test products and content across Holiday Extras and connected supplier test systems.

## How scenario values work

Every scenario is selected by one documented value: a country code, a location code, a set of product codes, a product token, or a booking reference. Take the value from the tables on this page and send it in the request.

- Other request fields, such as dates, customer details, and vehicle details, are accepted and validated but do not change which scenario you get.
- Responses never contain a value that selects another scenario. Location codes in the locations response, product tokens in search results, and amendment or cancellation tokens in quote responses are illustrative and do not select anything.
- Post-booking scenarios use the predefined booking references listed below. These are separate from the `booking_reference` returned when you create a sandbox booking.
- Sandbox covers UK parking in GBP with `en-GB` content. Other currencies, countries, and languages are not available in sandbox.

Any value that is not in this catalogue returns `422 Unprocessable Entity` with `code: "unprocessable_entity"` and an `errors` entry naming the field that did not match. See [Error handling](../errors.md#sandbox-scenario-not-matched).

## Locations

Use these values with `GET /v2/locations?product_types=parking&country_codes={value}`. The `country_codes` parameter is required in sandbox.

| Scenario | Scenario value | Expected result |
|---|---|---|
| UK locations | `GB` | Returns six UK airports: `LHR`, `BRS`, `MAN`, `STN`, `BHX`, `EDI` |
| No locations | `AQ` | Returns `200 OK` with an empty array |

The airports in the `GB` response are illustrative. Use the location codes in the next section to search.

## Parking search

Use these values as `location_code` with `GET /v2/products/parking` or `GET /v2/products/parking/detailed`. Send `location_type=iata` and `currency=GBP`. The detailed endpoint also needs `accept-language: en-GB`.

| Scenario | Scenario value | Expected result |
|---|---|---|
| Products available | `LGW` | Returns nine products covering every `parking_type`, with a mix of refundable, non-refundable, amendable, and non-amendable policies, and different `product_requirements` |
| No products | `ZZZ` | Returns `200 OK` with an empty array |

The dates, prices, and `product_token` values in the response are fixed and do not reflect the dates you send. The tokens do not select create-booking scenarios; use the product tokens in the next section instead.

## Product content

Use these values as repeated `product_codes` parameters with `GET /v2/content/parking` and `accept-language: en-GB`. Send the exact set of codes shown, in the order shown, for example `product_codes=FOO1&product_codes=FOO2&product_codes=FOO3`.

| Scenario | Scenario value | Expected result |
|---|---|---|
| Content for all products | `FOO1,FOO2,FOO3` | Returns content for all three products |
| Content for some products | `BAR1,BAR2,BAR3` | Returns content for `BAR1` and `BAR3` only. Use this to check how you handle a product with no content, for example a product that is no longer sold |
| No content | `BAZ1,BAZ2,BAZ3` | Returns `200 OK` with an empty array |

## Create booking scenarios

Use these values as `product_token` with `POST /v2/bookings/parking`. Send an `Idempotency-Key` header and a valid request body; the body does not change the outcome.

| Scenario | Scenario value | Expected result |
|---|---|---|
| Successful booking | `SBXBOOKING` | Returns `201 Created` with a booking reference and pricing |

### Price Lock and price validation

These scenarios return `409 Conflict` with `validated_price` and `alternative_product_token`. See [Price Lock](./07-price-lock.md) for the customer handling flow. The alternative token in the response is illustrative and does not select a scenario.

| Scenario | Scenario value | Expected result |
|---|---|---|
| Price increased above threshold | `SBXLOCKINCREASE` | The quoted price can no longer be accepted; `validated_price` carries the current price |
| Price Lock window expired | `SBXLOCKEXPIRED` | The Price Lock window has passed and the price has increased |
| Product no longer available | `SBXNOAVAIL` | The product cannot be booked; `validated_price` is `null` |

### Create booking error scenarios

| Scenario | Scenario value | Expected result |
|---|---|---|
| Duplicate booking | `SBXDUPLICATE` | Returns `409 Conflict`; a booking already exists for this request |
| Idempotency conflict | `SBXCONFLICT` | Returns `409 Conflict`; the request has already been fulfilled |
| Idempotency processing | `SBXPROCESSING` | Returns `425 Too Early`; the same request is still being processed |
| Booking failed | `SBXFAIL` | Returns `502 Bad Gateway` |
| Downstream timeout | `SBXTIMEOUT` | Returns `504 Gateway Timeout` |

## Get booking scenarios

Use these references with `GET /v2/bookings/parking/{ref}`.

| Scenario | Booking reference | Expected result |
|---|---|---|
| Confirmed booking | `SBXCONFIRMED` | An active booking with every access method, refundable and amendable policies, supplier fulfilment complete, and `before_travel` requirements still to collect |
| Pending supplier fulfilment | `SBXPENDING` | An active booking where `supplier_fulfilment.status` is `pending` and no supplier reference is available yet |
| All details collected | `SBXCOMPLETE` | An active booking with every product requirement already supplied |
| Supplier fulfilment failed | `SBXFAILED` | An active booking where `supplier_fulfilment.status` is `failed` |
| Not amendable, past dates | `SBXNONAMEND` | A booking for a past date where `policies.amendments.permitted` is `false` |
| Not refundable | `SBXNOREFUND` | An active booking with no refund tiers in `policies.refunds` |
| Cancelled booking | `SBXCANCELLED` | A booking with `booking_status: "cancelled"` |

## Amendment quote scenarios

Use these references with `PATCH /v2/bookings/parking/{ref}/amendments/quote`. Any valid request body gives the same result for a given reference.

| Scenario | Booking reference | Expected result |
|---|---|---|
| Amendment without a price change | `SBXNOCHANGE` | Returns a quote where `amended_total` equals `original_total` |
| Amendment with a price change | `SBXCHANGE` | Returns a quote where `amended_total` is higher than `original_total` |
| Booking not amendable | `SBXNOAMEND` | Returns `409 Conflict` |
| Booking already cancelled | `SBXCANCELLED` | Returns `409 Conflict` |

## Amendment confirm scenarios

Use these references with `POST /v2/bookings/parking/{ref}/amendments/confirm`. Send an `Idempotency-Key` header and an `amendment_token` in the body; the token value is not checked in sandbox.

| Scenario | Booking reference | Expected result |
|---|---|---|
| Amendment confirmed | `SBXSUCCESS` | Returns `200 OK` with `booking_status: "active"` and the original and amended totals |
| Amendment token expired | `SBXEXPIRED` | Returns `409 Conflict` with `code: "token_expired"` |
| Idempotency conflict | `SBXCONFLICT` | Returns `409 Conflict`; the request has already been fulfilled |
| Idempotency processing | `SBXTOOEARLY` | Returns `425 Too Early`; the same request is still being processed |
| Amendment failed | `SBXERROR` | Returns `502 Bad Gateway` |

## Cancellation quote scenarios

Use these references with `GET /v2/bookings/parking/{ref}/cancellations/quote`.

| Scenario | Booking reference | Expected result |
|---|---|---|
| Full refund | `SBXFULLREF` | Returns a quote with `is_refundable: true` and a refund equal to the booking total |
| Partial refund | `SBXPARTREF` | Returns a quote with `is_refundable: true` and a partial refund |
| No refund | `SBXNOREF` | Returns a quote with `is_refundable: false` and a zero refund. The booking can still be cancelled |
| Booking not cancellable | `SBXNOCANCEL` | Returns `409 Conflict` |
| Booking already cancelled | `SBXCANCELLED` | Returns `409 Conflict` |

## Cancellation confirm scenarios

Use these references with `POST /v2/bookings/parking/{ref}/cancellations/confirm`. Send an `Idempotency-Key` header and a `cancellation_token` in the body; the token value is not checked in sandbox.

| Scenario | Booking reference | Expected result |
|---|---|---|
| Cancellation confirmed | `SBXSUCCESS` | Returns `200 OK` with `booking_status: "cancelled"` and the refund amount |
| Cancellation token expired | `SBXEXPIRED` | Returns `409 Conflict` with `code: "token_expired"` |
| Idempotency conflict | `SBXCONFLICT` | Returns `409 Conflict`; the request has already been fulfilled |
| Idempotency processing | `SBXTOOEARLY` | Returns `425 Too Early`; the same request is still being processed |
| Cancellation failed | `SBXERROR` | Returns `502 Bad Gateway` |
