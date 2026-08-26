# POST /v2/bookings/parking/{ref}/amendments/confirm

Confirms an amendment using the `amendment_token` from a quote. The booking is updated immediately on success.

---

## Request

```http
POST https://api.holidayextras.com/partner-api/v2/bookings/parking/{ref}/amendments/confirm
```

### Headers

| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes | `Bearer {token}` - see [Authentication](../integration-guides/02-authentication.md) |
| `Content-Type` | Yes | `application/json` |
| `Idempotency-Key` | Yes | A unique key for this request, e.g. a UUID. Ensures the amendment is only applied once, even if the request is retried |

### Path parameters

| Parameter | Description |
|---|---|
| `ref` | The `booking_reference` of the booking being amended |

### Request body

```json
{
  "amendment_token": "amtoken_x9y8z7"
}
```

| Field | Type | Description |
|---|---|---|
| `amendment_token` | string | Token from the amendment quote response |

---

## Response

### 200 OK

Returns the updated booking status and amendment pricing.

```json
{
  "booking_status": "active",
  "pricing": {
    "original_total": {
      "amount_major": 89.99,
      "amount_minor": 8999,
      "currency": "GBP"
    },
    "amended_total": {
      "amount_major": 104.99,
      "amount_minor": 10499,
      "currency": "GBP"
    }
  }
}
```

### Response fields

| Field | Type | Description |
|---|---|---|
| `booking_status` | string | Updated booking status: `active` or `cancelled` |
| `pricing.original_total` | Money object | Price before the amendment |
| `pricing.amended_total` | Money object | Price after the amendment |

Money objects contain `amount_major`, `amount_minor`, and `currency`. Use `amount_major` for display and `amount_minor` for integer minor-unit handling.

---

## Error responses

Errors follow [RFC 9457 Problem Details](../errors.md).

| Status | When |
|---|---|
| `400 Bad Request` | Missing or invalid request body |
| `401 Unauthorized` | Missing or expired bearer token |
| `404 Not Found` | Booking reference does not exist |
| `409 Conflict` | Amendment token is no longer valid - a fresh quote gives you a new one |
| `409 Conflict` | Token does not match the booking reference |
| `409 Conflict` | Token type does not match this operation |
| `409 Conflict` | A confirm with this idempotency key already exists with a different request body |
| `500 Internal Server Error` | Unexpected server error |

---

## Sandbox examples

### Happy paths

#### Confirm an amendment

Run an amendment quote first to get an `amendment_token`, then confirm it here.

```bash
curl -X POST "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/{ref}/amendments/confirm" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: $(uuidgen)" \
  -d '{
    "amendment_token": "{token_from_quote}"
  }'
```

#### Replay - same idempotency key

Resending with the same `Idempotency-Key` returns the original response without applying the amendment again.

---

### Error scenarios

#### Expired amendment token (409)

Use a token whose `amendment_token_valid_until` has passed to trigger this response.

#### Wrong token type (409)

Pass a cancellation token rather than an amendment token to trigger this error.

---

## Related

- [PATCH /v2/bookings/parking/{ref}/amendments/quote](./patch-parking-amendment-quote.md) - request a quote before confirming
- [GET /v2/bookings/parking/{ref}](./get-parking-booking.md) - retrieve the updated booking after confirming
