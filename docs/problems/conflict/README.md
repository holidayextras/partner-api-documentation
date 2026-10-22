# Conflict

| Field | Value |
|-------|-------|
| **Status** | 409 |
| **Code** | `conflict` |

The request conflicts with the current state of the resource. Several causes share this type, so check `code` for the specific one.

## Common causes

- The quoted price can no longer be accepted when creating a booking (`code: price_changed`)
- An amendment or cancellation token has expired, does not match the booking, or is the wrong type (`token_expired`, `token_mismatch`, `wrong_token_type`, `invalid_token`)
- Reusing an idempotency key with a different request body. The same key with the same body returns the original result
- Attempting to cancel a booking that is already cancelled
- Concurrent modification of the same resource

## Example response

```json
{
  "type": "https://github.com/holidayextras/partner-api-documentation/tree/main/docs/problems/conflict",
  "title": "Conflict",
  "status": 409,
  "code": "conflict",
  "errors": [
    {
      "message": "Booking has already been cancelled"
    }
  ]
}
```

## How to resolve

For `price_changed`, follow the [price changed response](../../api-reference/post-parking-booking.md#price-changed-response) guidance and see [Price Lock](../../integration-guides/07-price-lock.md). For `token_expired`, `token_mismatch` or `invalid_token`, request a new quote to get a fresh token. For `wrong_token_type`, send the token to the matching confirm endpoint. Otherwise check the current state of the resource before retrying. If using idempotency keys, verify the original request completed successfully.
