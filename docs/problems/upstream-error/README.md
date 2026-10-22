# Upstream Error

| Field | Value |
|-------|-------|
| **Status** | `502`, or the upstream status when raised at the gateway |
| **Code** | `upstream_error` |

An upstream service returned an error that has been surfaced through the API. The HTTP status code on the response reflects the status returned by that upstream service. This is usually a transient, server-side error and can be retried.

## Common causes

- A downstream service returned a `502` without describing the failure
- The gateway received an error from the API that was not a problem details body

## Example response

```json
{
  "type": "https://github.com/holidayextras/partner-api-documentation/tree/main/docs/problems/upstream-error",
  "title": "Upstream Error",
  "status": 502,
  "code": "upstream_error",
  "trace_id": "abc123def456",
  "errors": []
}
```

## How to resolve

For `5xx` statuses, retry the request with exponential backoff (e.g. 2s, 4s, 8s) and stop after 3 attempts. If the error persists, contact support with the `trace_id` from the response. When retrying a write -- creating a booking, or confirming an amendment or cancellation -- send the **same** `Idempotency-Key` as the original request. A `4xx` status indicates a problem with the request itself -- review the request before retrying.
