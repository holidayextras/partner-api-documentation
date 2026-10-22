# Too Early

| Field | Value |
|-------|-------|
| **Status** | 425 |
| **Code** | `too_early` |

A request with the same `Idempotency-Key` is still being processed, so this one was not started.

## Common causes

- A booking create, amendment confirm or cancellation confirm was retried with the same `Idempotency-Key` before the original request finished

## Example response

```json
{
  "type": "https://github.com/holidayextras/partner-api-documentation/tree/main/docs/problems/too-early",
  "title": "Too Early",
  "status": 425,
  "code": "too_early",
  "errors": [],
  "detail": "Failed to create parking booking",
  "trace_id": "42da6b3d789063248c09a96c84e454e0"
}
```

## How to resolve

Wait a few seconds, then retry with the **same** `Idempotency-Key`. When the original request finishes, the retry returns its result. Use exponential backoff if the error persists.
