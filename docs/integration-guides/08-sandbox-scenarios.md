# Sandbox scenario catalogue

Use this catalogue when you need a consistent API response for a specific scenario.

Sandbox uses fake data. Each documented scenario returns a predictable response, allowing you to inspect it and check how your integration handles it.

Repeating the same documented request returns the same scenario. Use staging to test the complete end-to-end journey with test products and content across Holiday Extras and connected supplier test systems.

For create-booking scenarios, start with locations, run a parking search, use the product code to identify the scenario you want, then create the booking with the returned `product_token`.

For post-booking scenarios, use the predefined booking reference or token listed in the relevant table. These are separate from the `booking_reference` returned when you create a sandbox booking and are for sandbox only.

## Search and setup

| Scenario | Scenario value | Endpoint | Expected result |
|---|---|---|---|
| UK parking location scenarios | `country_codes=GB` | `GET /v2/locations?product_types=parking&country_codes=GB` | Returns sandbox location scenarios for the UK market |
| German parking location scenarios | `country_codes=DE` | `GET /v2/locations?product_types=parking&country_codes=DE` | Returns sandbox location scenarios for the German market |
| UK parking search catalogue | `currency=GBP` | `GET /v2/products/parking` | Returns UK sandbox products, one product per scenario |
| EUR parking search catalogue | `currency=EUR` | `GET /v2/products/parking` | Returns EUR sandbox products, including EUR mirrors where available |
| No availability | `location_code=XXX` | `GET /v2/products/parking` or `GET /v2/products/parking/detailed` | Returns `200 OK` with an empty array |
| English product content | `accept-language: en-GB` | `GET /v2/content/parking` | Returns English parking content |
| German product content | `accept-language: de-DE` | `GET /v2/content/parking` | Returns German parking content |

Additional IATA values are available in sandbox. UK IATA values such as `LTN`, `LGW`, `BHX`, `NCL`, `MAN`, `BRS`, and `EDI` select UK parking scenarios. EU IATA values such as `FRA`, `NUE`, `DUS`, `BER`, `LEJ`, `PMI`, `HAM`, `DTM`, and `FMM` select EU parking scenarios.

## Create booking scenarios

Search for the product code, then create the booking with that product's `product_token`.

| Scenario | GBP product code | EUR product code | Expected result |
|---|---|---|---|
| Standard successful booking and price validation | `SBXPOK` | `SBPOKE` | Booking is created successfully |
| Supplier reference access method | `SBXSR1` | `SBSR1E` | Created booking uses a supplier reference access method |
| Licence plate access method | `SBXLP1` | `SBLP1E` | Created booking uses the vehicle registration as the access method |
| Pending supplier fulfilment | `SBXPF1` | `SBPF1E` | Created booking has pending supplier fulfilment |
| Holiday Extras reference fallback | `SBXHX1` | - | Created booking falls back to the Holiday Extras reference when no supplier reference is available |
| Supplier confirmation access method | `SBXSC1` | `SBSC1E` | Created booking tells the customer to follow supplier confirmation instructions |
| Barcode access method | `SBXBC1` | - | Created booking uses a barcode access method |
| All partner access methods | `SBXAA1` | `SBAA1E` | Created booking includes all partner access methods available for testing |
| Partial content booking | `SBXPC1` | `SBPC1E` | Created booking has partial product content |
| EU flexible booking | `SBXE01` | `SBE01E` | Created booking is flexible in the EU scenario set |
| Downstream service unreachable on amend or cancel | `SBXAU1` | `SBAU1E` | Booking can be created, then amend or cancel operations hit downstream service unavailable behaviour |
| Downstream timeout on amend or cancel | `SBXAT1` | `SBAT1E` | Booking can be created, then amend or cancel operations hit downstream timeout behaviour |
| UK non-cancellable, amend-only booking | `SBXUC1` | `SBUC1E` | Booking can be amended but cancellation is not available |
| EU non-cancellable booking | `SBXEC1` | - | Booking is not cancellable in the EU scenario set |
| UK non-amendable booking | `SBXUA1` | `SBUA1E` | Booking does not allow amendments |
| EU non-amendable booking | `SBXEA1` | `SBEA1E` | Booking does not allow amendments in the EU scenario set |
| Already-cancelled booking for amend or cancel testing | `SBXCN1` | - | Created booking is already cancelled for follow-up flow testing |
| Date/time amendments not permitted | `SBXAN1` | `SBAN1E` | Created booking rejects date/time amendments |
| Non-flexible booking with customer and vehicle details still amendable | `SBXNX1` | `SBNX1E` | Customer and vehicle updates are allowed, price-affecting changes are rejected |

### Create booking error scenarios

| Scenario | Scenario value | Endpoint | Expected result |
|---|---|---|---|
| Duplicate booking detected | product code `SBXDUP` | `POST /v2/bookings/parking` | Returns a duplicate-booking error |
| Duplicate booking detected, EUR mirror | product code `SBDUPE` | `POST /v2/bookings/parking` | Returns a duplicate-booking error |
| UK downstream timeout | product code `SBXSVU` | `POST /v2/bookings/parking` | Returns a downstream timeout error |
| UK downstream timeout, EUR mirror | product code `SBSVUE` | `POST /v2/bookings/parking` | Returns a downstream timeout error |
| UK product unavailable | product code `SBXPUN` | `POST /v2/bookings/parking` | Returns a product-unavailable error |
| Idempotency key still processing | `Idempotency-Key: sbx-idempotency-processing` | `POST /v2/bookings/parking` | Returns an idempotency processing error with retry guidance |
| Idempotency key reused with different payload | `Idempotency-Key: sbx-idempotency-conflict` | `POST /v2/bookings/parking` | Returns an idempotency conflict |

## Price Lock and price validation

Search for the product code, then attempt to create the booking with that product's `product_token`. See [Price Lock](./07-price-lock.md) for the customer handling flow.

| Scenario | GBP product code | EUR product code | Expected result |
|---|---|---|---|
| Price validation successful | `SBXPOK` | `SBPOKE` | Booking continues with the quoted price |
| Price increased above threshold | `SBXPIR` | `SBPIRE` | Booking returns a price-change conflict where the quoted price can no longer be accepted |
| Price Lock window expired | `SBXPWE` | `SBPWEE` | Booking returns a Price Lock expired response |
| Product no longer available | `SBXPGN` | `SBPGNE` | Booking cannot continue because the selected product is no longer available |

## Get booking scenarios

Use these references with `GET /v2/bookings/parking/{ref}`.

| Scenario | GBP booking reference | EUR booking reference | Expected result |
|---|---|---|---|
| Default flexible booking | `SBX001` | - | Returns a default UK flexible booking |
| Golden-path booking | `SBXPRICEOK1` | `SBXPRICEOK1E` | Returns an active booking suitable for end-to-end amend and cancel testing |
| Supplier reference access method | `SBXSUPPREF1` | `SBXSUPPREF1E` | Returns a booking with supplier reference access |
| Licence plate access method | `SBXPLATE001` | `SBXPLATE001E` | Returns a booking with licence plate access |
| Barcode access method | `SBXBARCODE1` | `SBXBARCODE1E` | Returns a booking with barcode access |
| Supplier confirmation access method | `SBXCONFIRM1` | `SBXCONFIRM1E` | Returns a booking where supplier confirmation is the access method |
| All partner access methods | `SBXACCESS01` | `SBXACCESS01E` | Returns a booking with all partner access methods available |
| Pending supplier fulfilment | `SBXPENDING1` | `SBXPENDING1E` | Returns a booking where supplier fulfilment is pending |
| Holiday Extras reference fallback | `SBXHXFALLBK` | `SBXHXFALLBKE` | Returns a booking that falls back to the Holiday Extras reference |
| Partial content booking | `SBXPARTIAL1` | `SBXPARTIAL1E` | Returns a booking with partial product content |
| EU flexible booking | `SBXEU001` | `SBXEU001E` | Returns an EU flexible booking |
| Date/time amendments not permitted | `SBXNOAMEND1` | `SBXNOAMEND1E` | Returns a booking where date/time amendments are not permitted |
| Non-flexible booking | `SBXNONFLEX1` | `SBXNONFLEX1E` | Returns a booking where customer and vehicle details are amendable, but date changes are rejected |
| UK non-cancellable booking | `SBXUKNOCAN1` | `SBXUKNOCAN1E` | Returns a non-cancellable booking that can still be amended where allowed |
| EU non-cancellable booking | `SBXEUNOCAN1` | `SBXEUNOCAN1E` | Returns an EU non-cancellable booking |
| UK non-amendable booking | `SBXUKNOAM01` | `SBXUKNOAM01E` | Returns a UK booking that is not amendable |
| EU non-amendable booking | `SBXEUNOAM01` | `SBXEUNOAM01E` | Returns an EU booking that is not amendable |
| Cancelled booking | `SBXCANC001` | `SBXCANC001E` | Returns a cancelled booking |
| Booking not found | `SBXNOTFOUND` | - | Returns a not-found error |
| Downstream timeout | `SBXTIMEOUT1` | `SBXTIMEOUT1E` | Returns a downstream timeout error |
| Downstream service unreachable | `SBXUNAVAIL1` | `SBXUNAVAIL1E` | Returns a downstream service unavailable error |

## Amendment quote scenarios

Use these references with `PATCH /v2/bookings/parking/{ref}/amendments/quote`.

| Scenario | GBP scenario value | EUR scenario value | Request shape | Expected result |
|---|---|---|---|---|
| Default amendment quote | `SBXPRICEOK1` | `SBXPRICEOK1E` | Any valid amendable field | Returns an amendment quote |
| No-amendments booking, customer or vehicle update | `SBXNOAMEND1` | `SBXNOAMEND1E` | Customer or vehicle fields only | Returns an amendment quote |
| No-amendments booking, date/time change | `SBXNOAMEND1` | `SBXNOAMEND1E` | Parking or flight date/time fields | Returns a conflict because date/time amendments are not permitted |
| Non-flexible booking, customer or vehicle update | `SBXNONFLEX1` | `SBXNONFLEX1E` | Customer or vehicle fields only | Returns an amendment quote |
| Non-flexible booking, date/time change | `SBXNONFLEX1` | `SBXNONFLEX1E` | Parking or flight date/time fields | Returns an error because price-affecting amendments are rejected |
| Booking already cancelled | `SBXCANC001` | `SBXCANC001E` | Any amendment body | Returns a booking-already-cancelled error |
| UK booking not amendable | `SBXUKNOAM01` | `SBXUKNOAM01E` | Any amendment body | Returns a not-amendable error |
| EU booking not amendable | `SBXEUNOAM01` | `SBXEUNOAM01E` | Any amendment body | Returns a not-amendable error |
| Downstream timeout | `SBXTIMEOUT1` | `SBXTIMEOUT1E` | Any amendment body | Returns a downstream timeout error |
| Downstream service unreachable | `SBXUNAVAIL1` | `SBXUNAVAIL1E` | Any amendment body | Returns a downstream service unavailable error |

## Amendment confirm scenarios

Use these values with `POST /v2/bookings/parking/{ref}/amendments/confirm`.

| Scenario | Scenario value | Expected result |
|---|---|---|
| Default amendment confirmation | booking reference `SBXPRICEOK1` or `SBXPRICEOK1E` with a valid amendment token | Confirms the amendment |

## Cancellation quote scenarios

Use these references with `GET /v2/bookings/parking/{ref}/cancellations/quote`.

| Scenario | GBP booking reference | EUR booking reference | Expected result |
|---|---|---|---|
| Default cancellation quote | `SBXPRICEOK1` | `SBXPRICEOK1E` | Returns a cancellation quote |
| UK booking not cancellable | `SBXUKNOCAN1` | `SBXUKNOCAN1E` | Returns a not-cancellable error |
| EU booking not cancellable | `SBXEUNOCAN1` | `SBXEUNOCAN1E` | Returns a not-cancellable error |
| Booking already cancelled | `SBXCANC001` | `SBXCANC001E` | Returns a booking-already-cancelled error |
| Downstream timeout | `SBXTIMEOUT1` | `SBXTIMEOUT1E` | Returns a downstream timeout error |
| Downstream service unreachable | `SBXUNAVAIL1` | `SBXUNAVAIL1E` | Returns a downstream service unavailable error |

## Cancellation confirm scenarios

Use these values with `POST /v2/bookings/parking/{ref}/cancellations/confirm`.

| Scenario | Scenario value | Expected result |
|---|---|---|
| Default cancellation confirmation | booking reference `SBXPRICEOK1` or `SBXPRICEOK1E` with a valid cancellation token | Confirms the cancellation |

## Choosing scenarios for sign-off

For a typical airport parking integration, test at least:

- one successful search and booking using `SBXPOK` or `SBPOKE`
- one booking with licence plate access, such as `SBXLP1`
- one booking with a reference or supplier reference access method, such as `SBXSR1`
- one booking with barcode or supplier confirmation access, such as `SBXBC1` or `SBXSC1`
- one no-availability search using `location_code=XXX`
- one non-cancellable booking, such as `SBXUKNOCAN1`
- one non-amendable or non-flexible booking, such as `SBXUKNOAM01`, `SBXNOAMEND1`, or `SBXNONFLEX1`
- one idempotency conflict using `sbx-idempotency-conflict`
- one Price Lock or price validation exception using `SBXPIR`, `SBXPWE`, or `SBXPGN`
