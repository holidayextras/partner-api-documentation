# Too Many Requests

| Field | Value |
|-------|-------|
| **Status** | 429 |
| **Code** | `too_many_requests` |

The request rate limit for your credentials has been exceeded. The request was rejected before it was processed, so nothing was created or changed.

## Common causes

- Sustained request volume above the rate agreed for your account
- A burst of parallel requests, such as a batch job started all at once
- A retry loop with no backoff, which multiplies the original request rate

## Example response

```json
{
  "type": "https://github.com/holidayextras/partner-api-documentation/tree/main/docs/problems/too-many-requests",
  "title": "Too Many Requests",
  "status": 429,
  "detail": "Request rate limit exceeded. See the Retry-After header for wait time.",
  "code": "too_many_requests",
  "errors": []
}
```

## How to resolve

Wait at least the number of seconds in the `Retry-After` header, then retry with exponential backoff (e.g. 1s, 2s, 4s) and add jitter so parallel clients do not retry in lockstep. When retrying a write -- creating a booking, or confirming an amendment or cancellation -- send the **same** `Idempotency-Key` as the original request. Cap the number of concurrent in-flight requests rather than relying on retries alone. If your integration needs a higher limit, contact your account manager.
