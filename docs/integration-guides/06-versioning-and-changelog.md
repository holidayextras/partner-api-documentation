# Versioning and changelog

Partner API is versioned so you can build with confidence and plan changes before they affect customers.

All current Partner API endpoints are under `/v2/`. The OpenAPI schema describes the current request and response contract, and the documentation explains how to use that contract in a customer journey.

---

## What the API version means

The path version, such as `/v2/`, marks the major API contract. We use a new major version when a change would require partners to update their integration before continuing to use the API safely.

Within the current major version, partners always receive the latest backwards-compatible updates. These changes let partners adopt new fields, products, events, and behaviours when they are ready.

Where a full semantic version is surfaced by the API, it is returned in the `API-Version` response header, for example:

```http
API-Version: v2.1.3
```

---

## Change types

| Change type | What it means | What partners should do |
|---|---|---|
| Backwards-compatible change | Existing requests keep working, and existing response fields keep their meaning | No urgent action. Review the change and decide whether to use the new capability |
| Breaking change | Existing integrations may need a code or journey change to keep working safely | Plan and test the required change before the old behaviour is removed |
| Deprecation | A field, endpoint, enum value, or behaviour is still available, but is planned for removal or replacement | Move to the replacement path within the communicated timeline |
| Documentation-only change | The docs are clarified without changing API behaviour | Review if the clarification affects how your integration presents or handles the journey |

---

## Examples of backwards-compatible changes

Examples include, but are not limited to:

- adding an optional request field
- relaxing request validation, such as allowing `null` for a request field that previously accepted only a string
- adding a response field
- adding a new enum value
- adding a new product type
- adding a new access method
- adding a new webhook event type
- adding a new problem type or error code
- adding new fields inside `errors`
- adding a new endpoint under the same major version
- improving descriptions, examples, or OpenAPI metadata without changing behaviour
- naming a schema that was previously inlined, which can change the type names your client generator produces without changing the payload

Your integration should tolerate these changes where possible. For example, if a response includes an unknown enum value, keep the customer journey stable and log the value for review rather than failing the whole request.

---

## Examples of breaking changes

Breaking changes can include:

- removing or renaming an endpoint
- removing a response field that partners are expected to use
- changing the meaning or type of a field
- making an optional request field required
- removing an enum value
- changing authentication requirements
- changing idempotency behaviour
- changing a status code in a way that affects an existing success or error flow
- changing default behaviour or business meaning
- tightening validation so that a request that worked before is rejected
- changing webhook signature rules in a way that requires partner code changes
- changing a token flow in a way that prevents existing quote, booking, amendment, or cancellation journeys from continuing

When a breaking change is needed, Holiday Extras will follow the six-month deprecation and sunsetting process below. This gives affected partners time to build and test the replacement before the existing version or integration flow is retired.

---

## Deprecations

When something is deprecated, it remains available while partners move to the replacement. Holiday Extras' standard sunsetting period is six months once a fully functional replacement is available in production. During this period, the deprecated version or integration flow remains available and supported.

A deprecation notice should explain:

- what is being deprecated
- what replaces it
- which endpoints, fields, or behaviours are affected
- what action is needed
- when the old behaviour is expected to stop being supported

Holiday Extras will publish the deprecation and sunset dates when the replacement is released. We will send a reminder around two months into the period and a final reminder around one month before the sunset date. We will retire the deprecated version or integration flow at the end of the six-month period.

We may shorten this period for urgent security or compliance changes. If this happens, we will contact affected partners as soon as possible and explain the available mitigations.

For deprecated major versions, responses may include runtime headers that make the timeline easier to track in logs and monitoring:

- `Deprecation`: when deprecation started
- `Sunset`: when the old version is expected to reach end of life
- `Link`: where to find migration guidance

If a deprecated major version reaches end of life, it may return `410 Gone` with a link to the migration guidance. If a deprecation affects your integration, plan the change in staging before releasing it to production.

---

## Enum and field handling

Build your integration to handle additive changes safely:

- Treat unknown enum values as something to review, rather than a reason to fail the whole journey.
- Keep a sensible fallback for display labels where your UI maps API values to customer-facing text.
- Ignore response fields you do not use.
- Avoid strict response parsing that fails when a new optional field is added.
- Use the OpenAPI schema as the contract for known fields, but keep your parser tolerant of new fields.

This is especially useful for AI-assisted integrations and generated clients, where overly strict parsing can turn a small additive change into an avoidable integration issue.

---

## Webhook schema versioning

Webhook payloads include `schema_version`. Use it to understand which webhook payload shape you received.

The current `booking.impacted` payload uses schema version `2.0.0`. The webhook remains a signal: when you receive it, verify the signature, deduplicate the event, then fetch the latest booking with `GET /v2/bookings/parking/{ref}`.

For webhook details, see [Webhooks](./05-webhooks.md).

---

## OpenAPI schema and docs

The OpenAPI schema is the source of truth for request and response shapes. Use it to generate clients, validate payloads, and check field types.

The docs sit alongside the schema. They explain:

- how the endpoint fits into the customer journey
- which fields are important for product display, checkout, confirmation, amendments, and cancellations
- how to handle errors and operational edge cases
- which behaviours to test before launch

If you need help understanding how the API contract applies to your integration, contact your Holiday Extras partner contact.

---

## Staying up to date

Before releasing changes to production:

- check the latest OpenAPI schema
- review the relevant endpoint and integration guide pages
- test in sandbox where deterministic scenarios are available
- test in staging for live-like supplier behaviour and webhook delivery

---

## Related guides

- [API overview](./01-api-overview.md)
- [Error handling](../errors.md)
- [Webhooks](./05-webhooks.md)
- [Sandbox testing](./03-sandbox-testing.md)

---

Next: [API reference](../../README.md#api-reference)
