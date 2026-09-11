# GET /v2/bookings/parking/{ref}/cancellations/quote

Returns a cancellation quote including the refund amount and a `cancellation_token` to confirm it. The quote step gives the customer a chance to see exactly what they'll be refunded before committing.

---

## Request

```http
GET https://api.holidayextras.com/partner-api/v2/bookings/parking/{ref}/cancellations/quote
```

### Headers

| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes | `Bearer {token}` - see [Authentication](../integration-guides/02-authentication.md) |

### Path parameters

| Parameter | Description |
|---|---|
| `ref` | The `booking_reference` of the booking to cancel |

### Example request

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/SBXFULLREF/cancellations/quote" \
  -H "Authorization: Bearer {token}"
```

---

## Response

### 200 OK

```json
{
  "cancellation_token": "cantoken_p3q2r1",
  "cancellation_token_valid_until": "2026-05-20T15:00:00.000Z",
  "is_refundable": true,
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
| `cancellation_token` | string | Pass this to the confirm endpoint to complete the cancellation |
| `cancellation_token_valid_until` | UTC datetime | The token is valid until this time. If the customer takes longer to confirm, running the quote again gets a fresh one |
| `is_refundable` | boolean | Whether a refund is available for this cancellation |
| `pricing.refund` | Money object | Amount to be refunded. Zero if the product is non-refundable |

Money objects contain `amount_major`, `amount_minor`, and `currency`. Use `amount_major` for display and `amount_minor` for integer minor-unit handling.

---

## Error responses

Errors follow [RFC 9457 Problem Details](../errors.md).

| Status | When |
|---|---|
| `401 Unauthorized` | Missing or expired bearer token |
| `404 Not Found` | Booking reference does not exist |
| `409 Conflict` | Booking has already been cancelled |
| `500 Internal Server Error` | Unexpected server error |

---

## Sandbox examples

Sandbox scenarios are selected by the booking reference in the path. See the [Sandbox scenario catalogue](../integration-guides/08-sandbox-scenarios.md#cancellation-quote-scenarios).

### Happy paths

#### Full refund - `SBXFULLREF`

Returns `is_refundable: true` with a refund equal to the booking total.

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/SBXFULLREF/cancellations/quote" \
  -H "Authorization: Bearer {token}"
```

#### Partial refund - `SBXPARTREF`

Returns `is_refundable: true` with a partial refund. Test that you show the exact amount returned rather than the original price.

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/SBXPARTREF/cancellations/quote" \
  -H "Authorization: Bearer {token}"
```

#### No refund - `SBXNOREF`

Returns `is_refundable: false` with a zero refund. The booking can still be cancelled; test that the customer understands nothing will be refunded.

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/SBXNOREF/cancellations/quote" \
  -H "Authorization: Bearer {token}"
```

---

### Error scenarios

#### Booking not cancellable (409) - `SBXNOCANCEL`

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/SBXNOCANCEL/cancellations/quote" \
  -H "Authorization: Bearer {token}"
```

#### Already cancelled booking (409) - `SBXCANCELLED`

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/SBXCANCELLED/cancellations/quote" \
  -H "Authorization: Bearer {token}"
```

---

## Related

- [POST /v2/bookings/parking/{ref}/cancellations/confirm](./post-parking-cancellation-confirm.md) - confirm this quote using the cancellation token
- [GET /v2/bookings/parking/{ref}](./get-parking-booking.md) - check `policies.refunds` to understand the refund terms before quoting
