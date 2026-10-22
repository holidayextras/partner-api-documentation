# Service Unavailable

| Field | Value |
|-------|-------|
| **Status** | 503 |
| **Code** | `service_unavailable` |

The service is temporarily unable to handle the request. This is a transient, server-side condition and can be retried.

## Common causes

- A service is being deployed or restarted
- A downstream booking or content service reported itself unavailable

## Example response

```json
{
  "type": "https://github.com/holidayextras/partner-api-documentation/tree/main/docs/problems/service-unavailable",
  "title": "Service Unavailable",
  "status": 503,
  "detail": null,
  "code": "service_unavailable",
  "trace_id": "abc123def456",
  "errors": []
}
```

`detail` is always `null` and `errors` always empty on a `5xx` -- server-side failures are described by `code` and `trace_id`, not by a message.

## How to resolve

If the response carries a `Retry-After` header, wait at least that long before the first retry. Otherwise retry with exponential backoff (e.g. 2s, 4s, 8s). Stop after 3 attempts. If the error persists, contact support with the `trace_id` from the response. See also [Dependency Unavailable](../dependency-unavailable/README.md) and [Service Overloaded](../service-overloaded/README.md), the other `503`s this API can return. Load shedding always returns `code: service_overloaded`. Branch on `code`: when a downstream service shed the load, the `type` is `.../service-unavailable`.
