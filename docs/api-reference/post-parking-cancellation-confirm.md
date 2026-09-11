# POST /v2/bookings/parking/{ref}/cancellations/confirm

Confirms a cancellation using the `cancellation_token` from a quote. The booking is cancelled immediately on success.

---

## Request

```http
POST https://api.holidayextras.com/partner-api/v2/bookings/parking/{ref}/cancellations/confirm
```

### Headers

| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes | `Bearer {token}` - see [Authentication](../integration-guides/02-authentication.md) |
| `Content-Type` | Yes | `application/json` |
| `Idempotency-Key` | Yes | A unique key for this request, e.g. a UUID. Ensures the cancellation is only applied once, even if the request is retried |

### Path parameters

| Parameter | Description |
|---|---|
| `ref` | The `booking_reference` of the booking being cancelled |

### Request body

```json
{
  "cancellation_token": "cantoken_p3q2r1"
}
```

| Field | Type | Description |
|---|---|---|
| `cancellation_token` | string | Token from the cancellation quote response |

---

## Response

### 200 OK

```json
{
  "booking_status": "cancelled",
  "pricing": {
    "refund": {
      "amount_major": 89.99,
      "amount_minor": 8999,
      "currency": "GBP"
    }
  }
}
```

### Response fields

| Field | Type | Description |
|---|---|---|
| `booking_status` | string | Updated booking status. After a successful cancellation this will be `cancelled` |
| `pricing.refund` | Money object | Amount to be refunded |

Money objects contain `amount_major`, `amount_minor`, and `currency`. Use `amount_major` for display and `amount_minor` for integer minor-unit handling.

---

## Error responses

Errors follow [RFC 9457 Problem Details](../errors.md).

| Status | When |
|---|---|
| `400 Bad Request` | Missing or invalid request body |
| `401 Unauthorized` | Missing or expired bearer token |
| `404 Not Found` | Booking reference does not exist |
| `409 Conflict` | Cancellation token is no longer valid - a fresh quote gives you a new one |
| `409 Conflict` | Token does not match the booking reference |
| `409 Conflict` | Token type does not match this operation |
| `409 Conflict` | A confirm with this idempotency key already exists with a different request body |
| `500 Internal Server Error` | Unexpected server error |

---

## Sandbox examples

Sandbox scenarios are selected by the booking reference in the path. The `cancellation_token` must be present but its value is not checked in sandbox; any string works. See the [Sandbox scenario catalogue](../integration-guides/08-sandbox-scenarios.md#cancellation-confirm-scenarios).

### Happy paths

#### Cancellation confirmed - `SBXSUCCESS`

Returns `200 OK` with `booking_status: "cancelled"`.

```bash
curl -X POST "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/SBXSUCCESS/cancellations/confirm" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: $(uuidgen)" \
  -d '{
    "cancellation_token": "{token_from_quote}"
  }'
```

---

### Error scenarios

Same request with the booking reference shown.

| Scenario | Booking reference | Response |
|---|---|---|
| Cancellation token expired | `SBXEXPIRED` | `409 Conflict` with `code: "token_expired"` - ask the customer to quote again |
| Idempotency conflict | `SBXCONFLICT` | `409 Conflict` - the request has already been fulfilled |
| Idempotency processing | `SBXTOOEARLY` | `425 Too Early` - retry shortly |
| Cancellation failed | `SBXERROR` | `502 Bad Gateway` |

---

## Related

- [GET /v2/bookings/parking/{ref}/cancellations/quote](./get-parking-cancellation-quote.md) - request a quote before confirming
- [GET /v2/bookings/parking/{ref}](./get-parking-booking.md) - retrieve the updated booking after confirming
