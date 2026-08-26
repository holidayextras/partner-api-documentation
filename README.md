# Holiday Extras Partner API - Documentation

The Partner API lets you search, book, and manage travel ancillary products and sell them under your own brand. You pass us customer and travel details; we handle supplier relationships, confirmations, and fulfilment.

| Product | Status |
|---|---|
| Airport Parking | Available |
| Transfers | Coming soon |
| Lounges | Coming soon |
| Fast Track | Coming soon |
| Hotels | Coming soon |
| Hotels With Parking | Coming soon |

New to the platform, or evaluating an upgrade? See [Why Partner API 2026](./why-partner-api-2026.md).

---

## Getting started

These apply across all products.

| Environment | Base URL |
|---|---|
| Sandbox | `https://api-sandbox.holidayextras.com/partner-api` |
| Staging | `https://api-staging.holidayextras.com/partner-api` |
| Production | `https://api.holidayextras.com/partner-api` |

For the full route from sandbox access to launch, see [Getting started with Partner API 2026](./onboarding.md).

Start in sandbox. We'll work with you as you move through each environment and prepare to go live.

Sandbox uses fake data to return predictable responses for documented scenarios. See [Sandbox testing](./docs/integration-guides/03-sandbox-testing.md) for how to use it.

The live OpenAPI spec is available at [`https://api-staging.holidayextras.com/partner-api/v2/schema.json`](https://api-staging.holidayextras.com/partner-api/v2/schema.json).

**API version:** All endpoints are prefixed `/v2/`. The v1 API will be deprecated; the date is to be confirmed.

**Authentication:** OAuth 2.0 `client_credentials` flow. See [Authentication Integration Guide](./docs/integration-guides/02-authentication.md).

To get your `client_id` and `client_secret`, contact [partnerconnect@holidayextras.com](mailto:partnerconnect@holidayextras.com). These are scoped to your account. Keep your `client_secret` secure.

**Content type:** All requests and responses are `application/json`.

---

## Common endpoints

These endpoints apply across all products.

| Purpose | Method | Path |
|---|---|---|
| [Auth token](docs/api-reference/post-auth-token.md) | POST | `https://auth.holidayextras.com/oauth2/token` |
| [List locations](docs/api-reference/get-locations.md) | GET | `/v2/locations` |


---

## Integration guides

Technical documentation for developers building against the Partner API, covering authentication, endpoint flows, and end-to-end booking journeys.

| Guide | Description |
|---|---|
| [API Overview](docs/integration-guides/01-api-overview.md) | Base URLs, environments and key endpoints |
| [Authentication](docs/integration-guides/02-authentication.md) | OAuth 2.0 client credentials flow - getting and managing tokens |
| [Sandbox testing](docs/integration-guides/03-sandbox-testing.md) | How to use predictable sandbox scenarios and choose the responses to test |
| [Search endpoints](docs/integration-guides/04-search-endpoints.md) | When to use the detailed endpoint vs separate search and content endpoints |
| [Price Lock](docs/integration-guides/07-price-lock.md) | Carry selected prices through checkout and handle exceptional changes |
| [Webhooks](docs/integration-guides/05-webhooks.md) | Receive signed booking-impact notifications and refetch the latest booking state |
| [Error handling](docs/errors.md) | How to handle validation errors, retries, idempotency conflicts, expired tokens, and support diagnostics |
| [Versioning and changelog](docs/integration-guides/06-versioning-and-changelog.md) | How API versions, backwards-compatible changes, deprecations, and schema updates are handled |
| [Price Lock](docs/integration-guides/07-price-lock.md) | Carry selected prices through checkout and handle exceptional changes |
| [Sandbox scenario catalogue](docs/integration-guides/08-sandbox-scenarios.md) | Sandbox values for booking, access, amendment, cancellation, and Price Lock scenarios |

---

## Airport Parking

### API reference

| Purpose | Method | Path |
|---|---|---|
| [Search parking (with content)](docs/api-reference/get-parking-availability-detailed.md) | GET | `/v2/products/parking/detailed` |
| [Search parking](docs/api-reference/get-parking-availability.md) | GET | `/v2/products/parking` |
| [Parking content](docs/api-reference/get-parking-content.md) | GET | `/v2/content/parking` |
| [Create booking](docs/api-reference/post-parking-booking.md) | POST | `/v2/bookings/parking` |
| [Get booking](docs/api-reference/get-parking-booking.md) | GET | `/v2/bookings/parking/{ref}` |
| [Amendment quote](docs/api-reference/patch-parking-amendment-quote.md) | PATCH | `/v2/bookings/parking/{ref}/amendments/quote` |
| [Confirm amendment](docs/api-reference/post-parking-amendment-confirm.md) | POST | `/v2/bookings/parking/{ref}/amendments/confirm` |
| [Cancellation quote](docs/api-reference/get-parking-cancellation-quote.md) | GET | `/v2/bookings/parking/{ref}/cancellations/quote` |
| [Confirm cancellation](docs/api-reference/post-parking-cancellation-confirm.md) | POST | `/v2/bookings/parking/{ref}/cancellations/confirm` |

---

### User guides

Guidance on how to present and sell parking effectively, covering content, conversion best practices, and customer experience.

| Guide | Scope | Description |
|---|---|---|
| [Displaying Prices](docs/user-guides/displaying-prices.md) | All products | How to format and display prices correctly across currencies and locales |
| [Customer Journey](docs/user-guides/customer-journey.md) | Airport Parking | What customers need from search through post-booking updates |
| [Parking Integration Paths](docs/user-guides/integration-paths/index.md) | Airport Parking | How to shape the parking journey when selling standalone, alongside a wider travel booking, or alongside a flight |
| [Selling Parking](docs/user-guides/selling-parking.md) | Airport Parking | Product types, content fields to surface, and how to present parking to customers |
| [Accessing Parking](docs/user-guides/accessing-parking.md) | Airport Parking | How to deliver booking details, entry codes, and confirmation content to customers |

---

## FAQs

Other common questions are answered in our [FAQs page](./faqs.md).
