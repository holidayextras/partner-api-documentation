# Error handling

The OpenAPI schema describes the error response shape for each endpoint. This guide explains how to respond to those errors in your integration: when to retry, when to ask the customer to take action, what to show in your UI, and what to send Holiday Extras if you need support.

Partner API errors use RFC 9457 Problem Details. Each error response tells you what went wrong, but your integration should still decide how to handle it based on the status code, error type, and where the customer is in the journey.

---

## Error response shape

Most API errors use this shape:

```json
{
  "type": "https://docs.holidayextras.co.uk/partner/v2/problems/bad-request",
  "title": "Bad Request",
  "status": 400,
  "code": "bad_request",
  "trace_id": "7b2f9b0d6b8f4c8d9f2a123456789abc",
  "errors": [
    {
      "message": "must have required property 'location_code'",
      "field": "location_code",
      "code": "required"
    }
  ]
}
```

| Field | Description |
|---|---|
| `type` | Stable problem type URI. Use this, where available, to identify the kind of error |
| `title` | Short summary of the error |
| `status` | HTTP status code |
| `code` | Top-level machine-readable code summarising the response category, such as `bad_request` or `unprocessable_entity` |
| `trace_id` | Identifier Holiday Extras can use to investigate the request |
| `errors` | Validation or request-specific error details. This may be an empty array |

Items in `errors` usually include:

| Field | Description |
|---|---|
| `message` | Human-readable explanation of the specific error |
| `code` | Optional machine-readable code for the specific error, such as `required` |
| `field` | Optional field path related to the error. Some errors apply to the whole request rather than one field |

The OpenAPI schema remains the source of truth for the exact response object. This guide focuses on the partner behaviour that sits around it.

---

## General handling rules

| Status | Meaning | What your integration should usually do |
|---|---|---|
| `400 Bad Request` | The request is missing required fields or contains invalid values | Fix the request before retrying. If the error maps to customer input, show the relevant field message |
| `401 Unauthorized` | The bearer token is missing, invalid, or expired | Fetch a new token and retry once. If it still fails, log the error and investigate credentials |
| `403 Forbidden` | Your credentials are valid, but are not allowed to perform this action | Check account permissions or contact Holiday Extras |
| `404 Not Found` | The requested resource does not exist, or is not visible to your account | Show a useful not-found state. Check the booking reference, product code, or environment |
| `409 Conflict` | The request conflicts with the current booking, token, price, or idempotency state | Follow the endpoint-specific guidance and check what changed before retrying |
| `422 Unprocessable Entity` | The request is well formed, but cannot be completed in the current business state | Ask the customer to choose another option, refresh the quote, or restart the relevant flow |
| `429 Too Many Requests` | Too many requests were sent in a short period | Back off before retrying |
| `5xx` | A temporary or unexpected server problem | Retry when the operation is safe to retry. For write operations, use idempotency keys so the API can recognise the original action |

For customer-facing journeys, keep error messages calm and practical. Tell the customer what they can do next; keep technical details such as `trace_id`, raw problem type, and request payloads in your logs.

---

## Validation errors

Validation errors usually mean the request can be fixed without starting the whole journey again.

When `errors` is present, map each item back to the field in your form or integration payload. For example:

```json
{
  "type": "https://docs.holidayextras.co.uk/partner/v2/problems/bad-request",
  "title": "Bad Request",
  "status": 400,
  "code": "bad_request",
  "trace_id": "7b2f9b0d6b8f4c8d9f2a123456789abc",
  "errors": [
    {
      "field": "product_requirements.vehicle_registration",
      "message": "Vehicle registration is required for this product.",
      "code": "required"
    }
  ]
}
```

Recommended behaviour:

1. Keep the customer on the same step.
2. Highlight the affected field where possible.
3. Ask for the missing or corrected information.
4. Submit again only after the request has been corrected.

For customer-facing interfaces, use customer-friendly labels instead of raw API field names.

---

## Authentication errors

Partner API uses OAuth 2.0 `client_credentials`. Access tokens are valid for a limited time, so an occasional `401 Unauthorized` can be handled by refreshing the token.

Recommended behaviour:

1. Fetch a new token.
2. Retry the original request once.
3. If the retry also returns `401`, stop retrying and investigate credentials, environment, or account setup.

Cache the token until shortly before it expires. This keeps requests fast and avoids unnecessary calls to the auth endpoint.

---

## Idempotency conflicts

Write endpoints such as booking, amendment confirmation, and cancellation confirmation use `Idempotency-Key` to protect customers from duplicate actions when a request is retried.

If you send the same request again with the same idempotency key, the API returns the original result.

If you reuse the same idempotency key with a different request body, the API returns a conflict. Treat this as something to fix in your integration, not as a successful customer action.

Recommended behaviour:

| Situation | What to do |
|---|---|
| Same key, same body | Treat the response as the result of the original request |
| Same key, different body | Check the request before continuing. Generate a new key only if the customer is genuinely starting a new action |
| Network timeout after a write request | Retry with the same idempotency key so the API can recognise the original action |
| Customer changes the request before confirming | Use a new idempotency key for the new confirmed action |

Use one idempotency key per customer action. A fresh key for each booking, amendment, or cancellation keeps unrelated actions separate.

---

## Price and availability changes

Availability and price can change between search and booking, but price changes at booking that are not absorbed by [Price Lock](./integration-guides/07-price-lock.md) are expected to be exceptional. The search response returns a `product_token` and Price Lock details so you can continue through checkout with confidence.

When a booking is made within the Price Lock window, Holiday Extras absorbs eligible price movements. If the price has changed outside that window and the product is still available, the API will return the updated price and a fresh token wherever possible. Your integration can then show the latest price to the customer, or apply your own commercial handling in checkout.

Run a new search when the product is no longer available, or when the response does not include a fresh token for the customer to continue with.

Recommended behaviour:

| Error situation | What it means | What to do |
|---|---|---|
| Price changed since search | The product may still be available, but the quoted price is no longer current | Use the fresh token returned by the API where available, then show the updated price or absorb the difference |
| Product token expired | The token is no longer valid for booking | Run the search again to get a fresh token |
| Product no longer available | The supplier can no longer fulfil that product for the requested dates | Ask the customer to choose another product |

---

## Amendment and cancellation token errors

Amendments and cancellations use a quote-then-confirm flow. The quote step gives the customer the price or refund impact before they commit. The confirm step uses the token from the quote response.

Recommended behaviour:

| Error situation | What it means | What to do |
|---|---|---|
| Amendment token expired | The customer waited too long after getting the quote | Ask the customer to get a new quote |
| Cancellation token expired | The customer waited too long after getting the quote | Ask the customer to get a new cancellation quote |
| Wrong token type | An amendment token was used for cancellation, or a cancellation token was used for amendment | Correct the request before trying again |
| Amendment no longer permitted | The booking cannot be changed in that way now | Hide or block the unavailable change and explain the available options |
| Cancellation no longer available | The booking can no longer be cancelled, or no refund path applies | Show the latest cancellation outcome from the API |

Where possible, quote again rather than retrying a failed confirmation with the same expired or invalid token.

---

## Retry guidance

Retries are useful for temporary failures. The right retry pattern also protects customers from duplicate bookings, amendments, or cancellations.

Safe default:

- Retry `GET` requests when the failure looks temporary, such as a network error, `408`, `429`, or `5xx`.
- Retry write requests with the same `Idempotency-Key` so the API can return the original result instead of creating an unintentional duplicate.
- Change the request before retrying validation errors, permission errors, not-found errors, expired tokens, or booking-rule conflicts.
- Use exponential backoff for repeated retries. This gives temporary issues time to clear and reduces load on both systems.
- Keep customer-facing screens clear while retrying, so the customer understands whether their action is still being processed.

For webhooks, see the [Webhooks guide](./integration-guides/05-webhooks.md). Webhook delivery has its own retry rules.

---

## What to log for support

When raising an issue with Holiday Extras, include enough detail for us to find the request quickly.

Useful information:

- environment: sandbox, staging, or production
- endpoint and HTTP method
- approximate request time, including timezone
- `trace_id` from the error response, if present
- booking reference, if the request relates to a booking
- partner reference, if one was supplied
- idempotency key, for write requests
- webhook `Correlation-Id`, `Webhook-Id`, and `Webhook-Delivery-Id`, for webhook issues
- sanitised request body
- sanitised response body

Keep your `client_secret` out of support tickets.

---

## Endpoint-specific errors

The API reference pages list the status codes for each endpoint. Use those pages for endpoint-specific detail, then use this guide to decide how your integration should respond.

Common examples:

- [Create parking booking](./api-reference/post-parking-booking.md) explains price changes, expired product tokens, unavailable products, and idempotency conflicts.
- [Amendment quote](./api-reference/patch-parking-amendment-quote.md) explains unavailable amendments and invalid amendment requests.
- [Confirm amendment](./api-reference/post-parking-amendment-confirm.md) explains expired or invalid amendment tokens.
- [Cancellation quote](./api-reference/get-parking-cancellation-quote.md) explains unavailable cancellation quotes.
- [Confirm cancellation](./api-reference/post-parking-cancellation-confirm.md) explains expired or invalid cancellation tokens.

The OpenAPI schema remains the source of truth for exact fields and response shapes.

---

Next: [Versioning and changelog](./integration-guides/06-versioning-and-changelog.md)
