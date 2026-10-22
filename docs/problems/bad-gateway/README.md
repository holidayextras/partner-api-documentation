# Bad Gateway

| Field | Value |
|-------|-------|
| **Status** | 502 |
| **Code** | `bad_gateway` |

An upstream service returned an invalid response. This is a transient error and can be retried.

## Common causes

- A downstream booking system is temporarily unavailable
- Network issues between internal services

## Example response

```json
{
  "type": "https://github.com/holidayextras/partner-api-documentation/tree/main/docs/problems/bad-gateway",
  "title": "Bad Gateway",
  "status": 502,
  "detail": null,
  "code": "bad_gateway",
  "trace_id": "abc123def456",
  "errors": []
}
```

## How to resolve

Retry the request with exponential backoff (e.g. 2s, 4s, 8s). If the error persists after 3 attempts, contact support with the `trace_id`. When retrying a write -- creating a booking, or confirming an amendment or cancellation -- send the **same** `Idempotency-Key` as the original request.
