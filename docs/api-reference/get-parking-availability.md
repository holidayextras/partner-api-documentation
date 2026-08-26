# GET /v2/products/parking

Returns available parking products for a location and date range, with pricing only.

---

## Request

```http
GET https://api.holidayextras.com/partner-api/v2/products/parking
```

### Headers

| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes | `Bearer {token}` - see [Authentication](../integration-guides/02-authentication.md) |

### Query parameters

**Required**

| Parameter | Type | Description |
|---|---|---|
| `location_type` | string | Must be `iata` |
| `location_code` | string | Three-letter IATA airport code, e.g. `LHR` |
| `currency` | string | `GBP` or `EUR` |

**Optional**

| Parameter | Format | Description |
|---|---|---|
| `parking_entry_datetime` | `YYYY-MM-DDTHH:MM:SS` | When the customer expects to arrive at the car park, in airport local time |
| `parking_exit_datetime` | `YYYY-MM-DDTHH:MM:SS` | When the customer expects to leave the car park after returning to the airport, in airport local time |
| `outbound_departure_datetime` | `YYYY-MM-DDTHH:MM:SS` | Scheduled departure time of the customer's outbound flight from the airport where they will park |
| `outbound_flight_number` | string | Outbound flight number, e.g. `BA123`. Filters results to the correct terminal at multi-terminal airports |
| `inbound_arrival_datetime` | `YYYY-MM-DDTHH:MM:SS` | Scheduled arrival time of the customer's return flight back at the airport where they parked |
| `inbound_flight_number` | string | Inbound flight number, e.g. `BA456` |

### Specifying dates

You can provide dates in two ways, and they can be mixed - use whichever matches the information you have available on your customer's trip.

**By parking times** - pass `parking_entry_datetime` and `parking_exit_datetime` directly. Use this when your customer specifies when they want to drop off and collect their car.

```
parking_entry_datetime=2026-06-01T06:00:00
parking_exit_datetime=2026-06-15T18:00:00
```

**By flight times** - pass `outbound_departure_datetime` and `inbound_arrival_datetime`. Use this when your customer provides flight details rather than exact car park times. The API derives the parking window from the flights, giving the customer time to park before departure and collect their car after landing.

`inbound_arrival_datetime` is the return flight's scheduled arrival time at the airport where the customer parked their car. For example, if the customer parks at Gatwick and flies to Alicante, use the time their return flight lands back at Gatwick, not the time they arrive in Alicante.

```
outbound_departure_datetime=2026-06-01T08:30:00
inbound_arrival_datetime=2026-06-15T19:45:00
```

**Mixing the two** - each end of the parking window is independent, so the forms can be combined. Use this when you know one end precisely and only have a flight time for the other.

```
parking_entry_datetime=2026-06-01T06:00:00
inbound_arrival_datetime=2026-06-15T19:45:00
```

Each end of the window needs one of its two datetimes, so a single datetime on its own is rejected with a `400`. Every result returns the `parking_entry_datetime` and `parking_exit_datetime` actually used for pricing, whichever form you sent.

### Example request

By parking times:

```bash
curl "https://api.holidayextras.com/partner-api/v2/products/parking\
?location_type=iata\
&location_code=LGW\
&currency=GBP\
&parking_entry_datetime=2026-06-01T06:00:00\
&parking_exit_datetime=2026-06-15T18:00:00\
&outbound_flight_number=BA2490" \
  -H "Authorization: Bearer {token}"
```

By flight times:

```bash
curl "https://api.holidayextras.com/partner-api/v2/products/parking\
?location_type=iata\
&location_code=LGW\
&currency=GBP\
&outbound_departure_datetime=2026-06-01T08:30:00\
&inbound_arrival_datetime=2026-06-15T19:45:00\
&outbound_flight_number=BA2490\
&inbound_flight_number=BA2491" \
  -H "Authorization: Bearer {token}"
```

---

## Response

### 200 OK

Returns an array of available products.

```json
[
  {
    "code": "prk_lgw_001",
    "parking_entry_datetime": "2026-06-01T06:00:00",
    "parking_exit_datetime": "2026-06-15T18:00:00",
    "pricing": {
      "non_discounted": {
        "amount_major": 99.99,
        "amount_minor": 9999,
        "currency": "GBP"
      },
      "total": {
        "amount_major": 89.99,
        "amount_minor": 8999,
        "currency": "GBP"
      }
    },
    "product_requirements": [
      {
        "name": "vehicle_registration",
        "required_at": "at_booking"
      },
      {
        "name": "vehicle_make",
        "required_at": "before_travel"
      }
    ],
    "policies": {
      "amendments": {
        "permitted": true,
        "until": "2026-05-30T06:00:00.000Z"
      },
      "refunds": [
        {
          "amount": {
            "amount_major": 89.99,
            "amount_minor": 8999,
            "currency": "GBP"
          },
          "until": "2026-05-25T06:00:00.000Z"
        }
      ]
    },
    "product_token": "pgtoken_a1b2c3d4",
    "product_token_valid_until": "2026-06-01T00:00:00.000Z",
    "price_lock_valid_until": "2026-05-20T14:32:00.000Z"
  }
]
```

### Response fields

#### Per product

| Field | Type | Description |
|---|---|---|
| `code` | string | Product code - pass this to [GET /v2/content/parking](./get-parking-content.md) to fetch content |
| `parking_entry_datetime` | local datetime | Confirmed car park entry time used for pricing. If you searched by flight times, this is derived from the outbound departure time |
| `parking_exit_datetime` | local datetime | Confirmed car park exit time used for pricing. If you searched by flight times, this is derived from the inbound arrival time at the airport where the car is parked |
| `product_token` | string | Use this token to create the booking. It identifies the selected product, parking window, currency, and quoted price |
| `product_token_valid_until` | UTC datetime | The latest time this `product_token` can be used to attempt a booking |
| `price_lock_valid_until` | UTC datetime | The quoted `pricing.total` is protected against eligible price increases until this time. See [Price Lock](../integration-guides/07-price-lock.md) for how this works with product-token validity and exceptional changed-price responses. |

#### `pricing`

`pricing` contains Money objects. Use `amount_major` when displaying a price to customers. Use `amount_minor` when you need integer minor units for payment or reconciliation logic.

| Field | Type | Description |
|---|---|---|
| `non_discounted.amount_major` | number | Original price before any discount, in major units |
| `non_discounted.amount_minor` | integer | Original price before any discount, in minor units |
| `non_discounted.currency` | string | `GBP` or `EUR` |
| `total.amount_major` | number | Price to charge the customer, in major units |
| `total.amount_minor` | integer | Price to charge the customer, in minor units |
| `total.currency` | string | `GBP` or `EUR` |

#### `product_requirements`

Requirements that must be collected from the customer. Each entry has:

| Field | Description |
|---|---|
| `name` | The requirement, e.g. `vehicle_registration` |
| `required_at` | When it must be provided: `at_booking` means before confirming, `before_travel` means before travel. Use this to decide which fields belong in checkout and which can be collected later |

Search tells you whether each requirement is needed at booking or before travel. If a requirement is collected after booking, use [GET /v2/bookings/parking/{ref}](./get-parking-booking.md) to check its `requirement_deadline` before sending reminders or account-area prompts.

#### `policies`

| Field | Type | Description |
|---|---|---|
| `amendments.permitted` | boolean | Whether price-affecting amendments are permitted on the resulting booking. Customer and vehicle details can still be amended even when this is `false`. See the [amendment quote endpoint](./patch-parking-amendment-quote.md) for which fields are price-affecting. |
| `amendments.until` | UTC datetime or null | Deadline for price-affecting amendments. We recommend updating customer and vehicle details by the same cutoff to give suppliers time to prepare on the day. |
| `refunds` | array | Refund tiers - each entry has `amount` (a Money object) and `until` (UTC datetime). Empty if non-refundable |

---

## Error responses

Errors follow [RFC 9457 Problem Details](../errors.md).

| Status | When |
|---|---|
| `400 Bad Request` | Missing or invalid query parameters |
| `401 Unauthorized` | Missing or expired bearer token |
| `500 Internal Server Error` | Unexpected server error |

---

## Sandbox examples

### Happy paths

#### GBP - `LTN`

Returns a stable set of parking products priced in GBP.

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/products/parking\
?location_type=iata\
&location_code=LTN\
&currency=GBP\
&parking_entry_datetime=2026-06-01T06:00:00\
&parking_exit_datetime=2026-06-15T18:00:00" \
  -H "Authorization: Bearer {token}"
```

#### EUR - `FRA`

Returns a stable set of parking products priced in EUR.

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/products/parking\
?location_type=iata\
&location_code=FRA\
&currency=EUR\
&parking_entry_datetime=2026-06-01T06:00:00\
&parking_exit_datetime=2026-06-15T18:00:00" \
  -H "Authorization: Bearer {token}"
```

#### No availability - `XXX`

Returns a `200` with an empty array.

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/products/parking\
?location_type=iata\
&location_code=XXX\
&currency=GBP\
&parking_entry_datetime=2026-06-01T06:00:00\
&parking_exit_datetime=2026-06-15T18:00:00" \
  -H "Authorization: Bearer {token}"
```

---

### Validation errors

#### Entry datetime in the past

Triggers a `400` with a validation error.

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/products/parking\
?location_type=iata\
&location_code=LTN\
&currency=GBP\
&parking_entry_datetime=2020-01-01T06:00:00\
&parking_exit_datetime=2026-06-15T18:00:00" \
  -H "Authorization: Bearer {token}"
```

#### Exit before entry

Triggers a `400` when `parking_exit_datetime` is earlier than `parking_entry_datetime`.

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/products/parking\
?location_type=iata\
&location_code=LTN\
&currency=GBP\
&parking_entry_datetime=2026-06-15T18:00:00\
&parking_exit_datetime=2026-06-01T06:00:00" \
  -H "Authorization: Bearer {token}"
```

---

## Related

- [Search endpoints](../integration-guides/04-search-endpoints.md) - when to use this endpoint vs the detailed endpoint
- [GET /v2/locations](./get-locations.md) - get valid location codes to use in this request
- [GET /v2/products/parking/detailed](./get-parking-availability-detailed.md) - search with content included in a single response
- [GET /v2/content/parking](./get-parking-content.md) - fetch localised content for products returned by this endpoint
- [Selling Parking](../user-guides/selling-parking.md) - which content fields to surface and how
