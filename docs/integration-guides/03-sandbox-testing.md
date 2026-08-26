# Sandbox testing

Use sandbox to check that your integration handles known API responses correctly.

Sandbox uses fake data to return predictable responses. Each scenario has documented values, such as a location code, product code, or booking reference, so you can select the API response you want to test. This gives you a consistent way to test behaviour that can be difficult to reproduce elsewhere, including price changes, unavailable products, specific errors, access methods, and product or journey variations.

Use staging to test the complete end-to-end journey using test products and content across Holiday Extras and connected supplier test systems.

This page starts with a common sandbox approach, then lists the airport parking behaviours to cover. For the values used to select each scenario, see the [Sandbox scenario catalogue](./08-sandbox-scenarios.md).

---

## How sandbox scenarios work

Each scenario has three parts:

- **Scenario:** the API behaviour or response you want to test
- **How to select it:** the documented request value, product code, or booking reference used to identify the scenario
- **Expected result:** the consistent response or error the sandbox returns

For example, one sandbox location code selects a no-availability response. Product codes in the search response identify successful booking, price-change, and error scenarios. Repeating the same documented request returns the same scenario, keeping your tests consistent.

For airport parking:

1. Get an access token for sandbox. See [Authentication](../integration-guides/02-authentication.md).
2. Call the relevant discovery endpoint for the product you are integrating. For airport parking, start with `GET /v2/locations?product_types=parking`.
3. Run a product search. For airport parking, use [GET /v2/products/parking/detailed](../api-reference/get-parking-availability-detailed.md) or [GET /v2/products/parking](../api-reference/get-parking-availability.md).
4. For a create-booking scenario, use the product code in the [Sandbox scenario catalogue](./08-sandbox-scenarios.md) to identify the relevant product in the search response, then create the booking with its `product_token`.
5. For a post-booking scenario, use the predefined booking reference listed in the catalogue with the relevant booking, amendment, or cancellation endpoint.

The `booking_reference` returned when you create a sandbox booking identifies that booking. The predefined booking references in the scenario catalogue are a separate set used to select specific post-booking states.

---

## Core API responses to handle

You can cover each response in this table independently using the documented scenarios. Test the successful responses and variations that apply to your integration, then validate the complete end-to-end journey in staging.

| Step | Endpoint | What to check |
|---|---|---|
| Authenticate | `POST /oauth2/token` | Your integration can fetch, cache, and refresh a bearer token |
| Discover locations | `GET /v2/locations` | You can populate or validate a parking location from API data |
| Search parking | `GET /v2/products/parking/detailed` or `GET /v2/products/parking` | You can display available products, prices, policies, and required information |
| Create booking | `POST /v2/bookings/parking` | You can create a booking with an idempotency key, handle [Price Lock](./07-price-lock.md) behaviour, and store the returned booking reference |
| View booking | `GET /v2/bookings/parking/{ref}` | You can fetch the latest booking state and display customer-facing details |
| Amend booking | `PATCH /v2/bookings/parking/{ref}/amendments/quote`, then `POST /v2/bookings/parking/{ref}/amendments/confirm` | You can quote and confirm an allowed amendment, or hide the flow when amendments are not available |
| Cancel booking | `GET /v2/bookings/parking/{ref}/cancellations/quote`, then `POST /v2/bookings/parking/{ref}/cancellations/confirm` | You can quote and confirm an available cancellation and show any refund amount returned by the quote |

---

## Common scenario matrix

These scenarios apply across product types. The exact product and endpoint examples may vary as new products are added, but the integration behaviour is the same: read the policies and errors returned by the API, then show the right customer journey.

### Product flexibility

| Scenario | How to find it | What to check |
|---|---|---|
| Free cancellation | Search for a product with one or more refund tiers in `policies.refunds` | Refund terms are shown before booking, and cancellation quote/confirm flows are available after booking |
| Non-flexible product | Search for a product with no available refund tiers | Only the refund options that apply to the product are shown |
| Amendments permitted | Choose a product or booking where `policies.amendments.permitted` is `true` | Date/time amendment flows are available and use quote-before-confirm |
| Amendments not permitted | Choose or create a booking where `policies.amendments.permitted` is `false` | Price-affecting amendments are hidden or blocked gracefully |
| Customer details still amendable | Use a booking where price-affecting amendments are not permitted | Customer and vehicle detail updates are handled according to the fields returned in `amendable_data` |

### Operational and error handling

| Scenario | How to find it | What to check |
|---|---|---|
| Price changed since search | Use the Price Lock scenario values in the [Sandbox scenario catalogue](./08-sandbox-scenarios.md#price-lock-and-price-validation) | You can show the customer the validated price or absorb the difference when an alternative token is returned. See [Price Lock](./07-price-lock.md) for the full flow |
| Booking not found | Use `SBXNOTFOUND` from the [get booking scenarios](./08-sandbox-scenarios.md#get-booking-scenarios) | The customer or agent sees a useful not-found message |
| Cancellation not available | Use a non-cancellable booking from the [cancellation quote scenarios](./08-sandbox-scenarios.md#cancellation-quote-scenarios) | The customer sees that online cancellation is unavailable for this booking |
| Amendment token expired | Use the expired-token values from the [amendment confirm scenarios](./08-sandbox-scenarios.md#amendment-confirm-scenarios) | The customer is asked to quote the amendment again |
| Idempotency conflict | Use the idempotency scenario values in the [create booking error scenarios](./08-sandbox-scenarios.md#create-booking-error-scenarios) | Your integration treats the response as a request conflict, not as a successful duplicate |

---

## Airport parking scenario matrix

Use this matrix as a coverage checklist for airport parking-specific behaviour. The exact product you choose may vary; the important part is that your integration handles the response fields correctly.

### Required customer information

| Scenario | How to find it | What to check |
|---|---|---|
| Vehicle registration required | Search for a product whose requirements include `vehicle_registration` | Vehicle registration is collected at the right point in the customer journey |
| Email address required | Search for a product whose requirements include customer email | Email is collected and submitted in the expected booking field |
| Mobile number required | Search for a product whose requirements include `mobile_number` | Mobile number collection, validation, and submission work correctly |
| Flight details required | Search for a product whose requirements include flight number or terminal fields | Flight details are collected only when needed and submitted in the expected fields |
| Details required before travel | Search for a product with `required_at: "before_travel"`, then view the booking to check `product_requirements[].requirement_deadline` | Follow-up messaging or account-area prompts use the requirement deadline returned on the booking |

### Product display and content

| Scenario | How to find it | What to check |
|---|---|---|
| Park and ride | Search for a product with `parking_type` for park and ride | Transfer duration, transfer frequency, and arrival instructions are presented clearly |
| Meet and greet | Search for a product with `parking_type` for meet and greet | Terminal handover instructions are clear before the customer travels |
| Park and stroll | Search for a product with `parking_type` for park and stroll | Walking distance and directions are clear enough for the customer to choose confidently |
| Electric charging available | Search for product content that indicates electric charging | Customers can see whether electric charging is available |
| Larger vehicles | Search for products with vehicle size restrictions or larger vehicle messaging | Customers can understand whether their vehicle is suitable before booking |
| Leave keys / keep keys | Use product content fields that describe key handling | Key-handling expectations are shown before purchase and in confirmation messaging |

### Parking access methods

| Scenario | How to find it | What to check |
|---|---|---|
| Barcode | View a booking where `access_methods.type` includes `barcode` | Barcode values are rendered in a usable format when present |
| Licence plate recognition | View a booking where `access_methods.type` includes `licence_plate` | The customer sees the vehicle registration that will be used for access |
| Reference | View a booking where `access_methods.type` includes `reference` | The exact reference returned by the API is displayed prominently |
| Supplier confirmation | View a booking where `access_methods.type` includes `supplier_confirmation` | Customers are directed to follow the supplier's confirmation instructions |

---

## Sandbox scenario coverage checklist

When you are ready to move beyond sandbox, share examples of how your integration handles the agreed scenarios with your Holiday Extras partnerships contact.

Use the [Sandbox scenario catalogue](./08-sandbox-scenarios.md) when you need a specific booking state, access method, policy, or error response.

Depending on the scope of your integration, helpful examples include:

- An availability response and the resulting product display using sandbox data
- Handling of a successful booking response
- A customer confirmation built from the returned booking fields
- Handling of amendment quote and confirmation responses, if you support amendments
- Handling of cancellation quote and confirmation responses, if you support cancellations
- Examples showing how you handle unavailable amendments, unavailable refunds, missing required details, and different access methods

---

## What to test in staging

After checking predictable API responses in sandbox, use staging to validate the complete end-to-end journey. Staging uses test products and content and connects to supplier test systems.

- Search, booking, amendment, and cancellation flows across connected systems
- The customer journey using staging test products and content
- Webhook delivery, which is only available to test in staging

For the full onboarding path, see [Getting started with Partner API 2026](../../onboarding.md).

---

Next: [Search endpoints](./04-search-endpoints.md)
