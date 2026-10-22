# Problem Types

This page is the reference for the problem `type` URIs the API returns. For guidance on how your integration should respond to each one -- when to retry, what to show the customer -- see the [error handling guide](../errors.md).

Every error returned by Partner API v2 uses the same problem details body, served as `application/problem+json`:

```json
{
  "type": "https://github.com/holidayextras/partner-api-documentation/tree/main/docs/problems/conflict",
  "title": "Conflict",
  "status": 409,
  "detail": "The amendment token has expired",
  "code": "token_expired",
  "trace_id": "abc123def456",
  "errors": []
}
```

| Field | Description |
|-------|-------------|
| `type` | A stable URI identifying the class of problem. It resolves to the page for that problem, listed below. |
| `title` | A short, human-readable summary of the problem class. |
| `status` | The HTTP status code, repeated in the body. |
| `detail` | A human-readable explanation of this specific occurrence. `null` on a `5xx` raised by the API itself; a `5xx` raised at the gateway carries a short generic message. Do not branch on it. |
| `code` | A machine-readable value for this specific occurrence. Branch on this, not on `title` or `detail`. |
| `trace_id` | The identifier to quote when contacting support. Absent on errors raised at the gateway, which include every `401`, `403` and `429` caused by credentials or rate limits. |
| `errors` | Field-level failures, each with `field`, `message` and an optional `code`. Always empty on a `5xx`. |

## `type` and `code` are not the same thing

`type` names the class of problem and is drawn from the fixed list below. `code` names the specific occurrence and can be more precise than the `type`. In the example above, an expired amendment token is a conflict, so `type` is `.../conflict`, but `code` is the narrower `token_expired`.

Treat `type` as the stable link to documentation and `code` as the value your integration switches on. A new `code` can appear within an existing `type` without the `type` changing, so match `code` values you know and fall back on `status` for the rest rather than failing closed on an unrecognised value.

## Problem types

| Status | Canonical `code` | Problem |
|--------|------------------|---------|
| 400 | `bad_request` | [Bad Request](./bad-request/README.md) |
| 401 | `unauthorized` | [Unauthorized](./unauthorized/README.md) |
| 403 | `forbidden` | [Forbidden](./forbidden/README.md) |
| 404 | `resource_not_found` | [Resource Not Found](./resource-not-found/README.md) |
| 405 | `method_not_allowed` | [Method Not Allowed](./method-not-allowed/README.md) |
| 406 | `not_acceptable` | [Not Acceptable](./not-acceptable/README.md) |
| 408 | `request_timeout` | [Request Timeout](./request-timeout/README.md) |
| 409 | `conflict` | [Conflict](./conflict/README.md) |
| 410 | `gone` | [Gone](./gone/README.md) |
| 415 | `unsupported_media_type` | [Unsupported Media Type](./unsupported-media-type/README.md) |
| 422 | `unprocessable_entity` | [Unprocessable Entity](./unprocessable-entity/README.md) |
| 425 | `too_early` | [Too Early](./too-early/README.md) |
| 429 | `too_many_requests` | [Too Many Requests](./too-many-requests/README.md) |
| 500 | `internal_error` | [Internal Server Error](./internal-error/README.md) |
| 502 | `bad_gateway` | [Bad Gateway](./bad-gateway/README.md) |
| 502, or the upstream status | `upstream_error` | [Upstream Error](./upstream-error/README.md) |
| 503 | `service_unavailable` | [Service Unavailable](./service-unavailable/README.md) |
| 503 | `service_overloaded` | [Service Overloaded](./service-overloaded/README.md) |
| 503 | `dependency_unavailable` | [Dependency Unavailable](./dependency-unavailable/README.md) |
| 504 | `gateway_timeout` | [Gateway Timeout](./gateway-timeout/README.md) |

## Retrying

`4xx` problems describe something about the request and will fail again unchanged -- fix the request rather than retrying it. The exceptions are [Request Timeout](./request-timeout/README.md) and [Too Many Requests](./too-many-requests/README.md), where the request was never processed, and [Too Early](./too-early/README.md), where an earlier request with the same `Idempotency-Key` is still in progress. A later attempt can succeed.

`5xx` problems are server-side and usually transient. If the response carries a `Retry-After` header, wait at least that long before the first retry. Otherwise retry with exponential backoff (e.g. 2s, 4s, 8s). Stop after 3 attempts, then contact support with the `trace_id`.

When retrying a write -- creating a booking, or confirming an amendment or cancellation -- always send the **same** `Idempotency-Key` as the original request. A retry with a fresh key can duplicate an operation that had already succeeded.
