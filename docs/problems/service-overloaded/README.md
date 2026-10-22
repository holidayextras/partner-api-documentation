# Service Overloaded

| Field | Value |
|-------|-------|
| **Status** | 503 |
| **Code** | `service_overloaded` |

The service is temporarily shedding load to protect itself and did not process the request. This is a transient, server-side condition and can be retried. When the API itself sheds load, the response includes a `Retry-After` header giving the number of seconds to wait. When a downstream service shed the load, the `type` is [Service Unavailable](../service-unavailable/README.md) with this `code`, and there is no `Retry-After`.

## Common causes

- A spike in traffic across the platform
- The service is recovering from a period of high load

## Example response

```json
{
  "type": "https://github.com/holidayextras/partner-api-documentation/tree/main/docs/problems/service-overloaded",
  "title": "Service Overloaded",
  "status": 503,
  "detail": null,
  "code": "service_overloaded",
  "trace_id": "abc123def456",
  "errors": []
}
```

`detail` is always `null` and `errors` always empty on a `5xx` -- server-side failures are described by `code` and `trace_id`, not by a message.

## How to resolve

If the response carries a `Retry-After` header, wait at least that long; otherwise start at 2s. Then back off exponentially with jitter and stop after 3 attempts. When retrying a write, send the **same** `Idempotency-Key` as the original request. If the error persists, contact support with the `trace_id` from the response.
