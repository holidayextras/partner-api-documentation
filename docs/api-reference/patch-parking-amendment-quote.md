# PATCH /v2/bookings/parking/{ref}/amendments/quote

Returns a quote for amending a booking and an `amendment_token` to confirm it. The quote step gives the customer a chance to review any price change before committing.

---

## Request

```http
PATCH https://api.holidayextras.com/partner-api/v2/bookings/parking/{ref}/amendments/quote
```

### Headers

| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes | `Bearer {token}` - see [Authentication](../integration-guides/02-authentication.md) |
| `Content-Type` | Yes | `application/json` |

### Path parameters

| Parameter | Description |
|---|---|
| `ref` | The `booking_reference` of the booking to amend |

### Request body

Include only the fields you want to change. Fields not listed in the tables below are not supported on this endpoint. The `customer` object is partial on amendments: send only the customer fields being amended, and omitted customer fields are left unchanged.

Not all fields behave the same way for every product. There are two groups:

**Price-affecting fields:** changing these recalculates the price. They are only allowed when `policies.amendments.permitted` is `true` on the booking. Submitting any of these when `policies.amendments.permitted` is `false` returns a `422`.

| Field | Type | Description |
|---|---|---|
| `parking_entry_datetime` | `YYYY-MM-DDTHH:MM:SS` | Updated time the customer expects to arrive at the car park, in airport local time |
| `parking_exit_datetime` | `YYYY-MM-DDTHH:MM:SS` | Updated time the customer expects to leave the car park after returning to the airport, in airport local time |
| `outbound_departure_datetime` | `YYYY-MM-DDTHH:MM:SS` | Updated scheduled departure time of the customer's outbound flight from the airport where they parked |
| `inbound_arrival_datetime` | `YYYY-MM-DDTHH:MM:SS` | Updated scheduled arrival time of the customer's return flight back at the airport where they parked |

**Always-amendable fields:** these never affect the price and can be submitted on any booking within the `policies.amendments.until` window, even when `policies.amendments.permitted` is `false`.

| Field | Type | Description |
|---|---|---|
| `customer.given_name` | string | Updated customer first name |
| `customer.family_name` | string | Updated customer last name |
| `customer.email` | string | Updated customer email address |
| `product_requirements.vehicle_registration` | string | Updated vehicle registration |
| `product_requirements.outbound_flight_number` | string | Updated outbound flight number |
| `product_requirements.inbound_flight_number` | string | Updated inbound flight number |
| `product_requirements.mobile_number` | string | Updated mobile number |
| `product_requirements.vehicle_make` | string | Updated vehicle make |
| `product_requirements.vehicle_model` | string | Updated vehicle model |
| `product_requirements.vehicle_colour` | string | Updated vehicle colour |

For the smoothest experience on the day, submit all amendments by the end of the day before travel. This gives suppliers time to make sure everything is in place before the customer arrives.

A useful rule of thumb: fields that formed part of the original availability search (parking times or flight times) are price-affecting. Everything else is always amendable.

When amending by flight times, `inbound_arrival_datetime` should be the return flight's scheduled arrival time at the airport where the customer parked their car. The API uses it to derive the updated parking exit time.

You can mix always-amendable fields in a single request. You can also combine always-amendable and price-affecting fields, but if `policies.amendments.permitted` is `false`, the whole request is rejected. Submit them separately if needed.

```json
{
  "parking_entry_datetime": "2026-06-02T06:00:00",
  "parking_exit_datetime": "2026-06-16T18:00:00"
}
```

---

## Response

### 200 OK

```json
{
  "amendment_token": "amtoken_x9y8z7",
  "amendment_token_valid_until": "2026-05-20T15:00:00.000Z",
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
| `amendment_token` | string | Pass this to the confirm endpoint to apply the amendment |
| `amendment_token_valid_until` | UTC datetime | The token is valid until this time. If the customer takes longer to confirm, running the quote again gets a fresh one |
| `pricing.original_total` | Money object | Current booking price |
| `pricing.amended_total` | Money object | Price after the amendment |

Money objects contain `amount_major`, `amount_minor`, and `currency`. Use `amount_major` for display and `amount_minor` for integer minor-unit handling.

---

## Error responses

Errors follow [RFC 9457 Problem Details](../errors.md).

| Status | When |
|---|---|
| `400 Bad Request` | Missing or invalid request body fields |
| `401 Unauthorized` | Missing or expired bearer token |
| `404 Not Found` | Booking reference does not exist |
| `409 Conflict` | The amendment window has closed. Check `policies.amendments.until` on the booking for the deadline |
| `422 Unprocessable Entity` | One or more submitted fields are price-affecting and not permitted when `policies.amendments.permitted` is `false`. See field groupings above |
| `422 Unprocessable Entity` | No amendable fields were submitted |
| `500 Internal Server Error` | Unexpected server error |

---

## Sandbox examples

### Happy paths

#### Parking time change (amendments permitted)

```bash
curl -X PATCH "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/{ref}/amendments/quote" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "parking_entry_datetime": "2026-06-02T06:00:00",
    "parking_exit_datetime": "2026-06-16T18:00:00"
  }'
```

#### Flight time change (amendments permitted)

Use this when the customer's flight times have changed and you want the API to derive the updated parking window.

```bash
curl -X PATCH "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/{ref}/amendments/quote" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "outbound_departure_datetime": "2026-06-02T08:30:00",
    "inbound_arrival_datetime": "2026-06-16T19:45:00"
  }'
```

#### Vehicle registration update where parking time amendments are not permitted - `SBXNOAMEND1`

Returns `200 OK` with an `amendment_token`. This lets you test updating allowed product requirement data on a booking that does not permit date/time amendments.

```bash
curl -X PATCH "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/SBXNOAMEND1/amendments/quote" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "product_requirements": {
      "vehicle_registration": "XY21 ZAB"
    }
  }'
```

#### Customer details update when `policies.amendments.permitted` is `false` - `SBXNONFLEX1`

Always-amendable fields (customer details, vehicle details) go through even when `policies.amendments.permitted` is `false`.

```bash
curl -X PATCH "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/SBXNONFLEX1/amendments/quote" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "customer": {
      "given_name": "Jane",
      "family_name": "Smith"
    },
    "product_requirements": {
      "vehicle_registration": "XY21 ZAB"
    }
  }'
```

---

### Error scenarios

#### Price-affecting amendment when `policies.amendments.permitted` is `false` (422) - `SBXNONFLEX1`

Parking time and flight time changes are not permitted when `policies.amendments.permitted` is `false`.

```bash
curl -X PATCH "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/SBXNONFLEX1/amendments/quote" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "parking_entry_datetime": "2026-06-02T06:00:00",
    "parking_exit_datetime": "2026-06-16T18:00:00"
  }'
```

#### Parking time change where parking time amendments are not permitted (409) - `SBXNOAMEND1`

```bash
curl -X PATCH "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/SBXNOAMEND1/amendments/quote" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "parking_entry_datetime": "2026-06-02T06:00:00",
    "parking_exit_datetime": "2026-06-16T18:00:00"
  }'
```

#### Parking time and vehicle registration change where parking time amendments are not permitted (409) - `SBXNOAMEND1`

The booking accepts a registration-only quote, but this mixed request is rejected because it also includes a date/time amendment.

```bash
curl -X PATCH "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/SBXNOAMEND1/amendments/quote" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "parking_entry_datetime": "2026-06-02T06:00:00",
    "parking_exit_datetime": "2026-06-16T18:00:00",
    "product_requirements": {
      "vehicle_registration": "XY21 ZAB"
    }
  }'
```

#### No amendable fields submitted (422)

```bash
curl -X PATCH "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/{ref}/amendments/quote" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{}'
```

---

## Related

- [POST /v2/bookings/parking/{ref}/amendments/confirm](./post-parking-amendment-confirm.md) - confirm this quote using the amendment token
- [GET /v2/bookings/parking/{ref}](./get-parking-booking.md) - check `policies.amendments` before quoting
