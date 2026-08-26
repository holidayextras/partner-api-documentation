# GET /v2/bookings/parking/{ref}

Returns the full details of a parking booking.

---

## Request

```http
GET https://api.holidayextras.com/partner-api/v2/bookings/parking/{ref}
```

### Headers

| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes | `Bearer {token}` - see [Authentication](../integration-guides/02-authentication.md) |
| `accept-language` | Yes | Language for content fields in the response. One of: `en-GB`, `de-DE`, `nl-NL`, `fr-FR`, `es-ES`, `it-IT`, `pt-PT`, `pl-PL`, `cy-GB` |

### Path parameters

| Parameter | Description |
|---|---|
| `ref` | The `booking_reference` returned when the booking was created |

### Example request

```bash
curl "https://api.holidayextras.com/partner-api/v2/bookings/parking/GSFWRJ" \
  -H "Authorization: Bearer {token}" \
  -H "accept-language: en-GB"
```

---

## Response

### 200 OK

```json
{
  "booking_reference": "GSFWRJ",
  "booking_status": "active",
  "partner_reference": "YOUR-REF-001",
  "supplier_reference": "SUP-123456",
  "product_code": "LPH4",
  "parking_entry_datetime": "2026-06-01T06:00:00",
  "parking_exit_datetime": "2026-06-15T18:00:00",
  "customer": {
    "given_name": "Jane",
    "family_name": "Smith",
    "email": "jane.smith@example.com"
  },
  "product_requirements": [
    {
      "name": "vehicle_registration",
      "value": "AB12 CDE",
      "requirement_deadline": null
    },
    {
      "name": "vehicle_make",
      "value": null,
      "requirement_deadline": "2026-05-29T06:00:00.000Z"
    },
    {
      "name": "vehicle_colour",
      "value": null,
      "requirement_deadline": "2026-05-29T06:00:00.000Z"
    }
  ],
  "pricing": {
    "total": {
      "amount_major": 89.99,
      "amount_minor": 8999,
      "currency": "GBP"
    },
    "commission": {
      "amount_major": 12,
      "amount_minor": 1200,
      "currency": "GBP"
    },
    "commission_vat": {
      "amount_major": 2.4,
      "amount_minor": 240,
      "currency": "GBP"
    }
  },
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
  "supplier_fulfilment": {
    "status": "fulfilled",
    "expected_completion_datetime": null
  },
  "access_methods": [
    {
      "priority": 1,
      "type": "licence_plate",
      "value": "AB12 CDE"
    },
    {
      "priority": 2,
      "type": "reference",
      "value": "SUP-123456"
    }
  ],
  "confirmation_page_link": "https://www.holidayextras.com/confirmation/GSFWRJ",
  "product_content": {
    "name": "Long Stay South",
    "language": "en-GB",
    "terminal": "South",
    "parking_type": "park_and_ride",
    "distance_miles": 1.2,
    "transfer_duration_minutes": 8,
    "transfer_frequency_minutes": 10,
    "..." : "..."
  },
  "amendable_data": {
    "parking_entry_datetime": "2026-06-01T06:00:00",
    "parking_exit_datetime": "2026-06-15T18:00:00",
    "customer": {
      "given_name": "Jane",
      "family_name": "Smith",
      "email": "jane.smith@example.com"
    },
    "product_requirements": {
      "vehicle_registration": "AB12 CDE",
      "outbound_flight_number": "BA123"
    }
  }
}
```

### Response fields

| Field | Type | Description |
|---|---|---|
| `booking_reference` | string | The Holiday Extras booking reference |
| `booking_status` | string | Status of the booking with Holiday Extras: `active` or `cancelled`. Independent of the supplier fulfilment state - see `supplier_fulfilment` below |
| `partner_reference` | string or null | Your reference for this booking |
| `supplier_reference` | string or null | The supplier's own reference for the booking, when available. When present, this may also appear as a `reference` access method |
| `product_code` | string | The booked product code |
| `parking_entry_datetime` | local datetime | Booked entry time |
| `parking_exit_datetime` | local datetime | Booked exit time |
| `confirmation_page_link` | URI or null | URL to the Holiday Extras hosted confirmation page |
| `access_methods` | array | Preferred ordered list of ways the customer can access the car park. See subsection below for behaviour |

#### `supplier_fulfilment`

| Field | Type | Description |
|---|---|---|
| `status` | string | Supplier fulfilment state: `pending` (supplier has not yet confirmed), `fulfilled` (supplier has confirmed and supplier-side details are in place), or `failed` (supplier fulfilment could not be completed - by default Holiday Extras handles the rebook on your behalf) |
| `expected_completion_datetime` | UTC datetime or null | The supplier's estimated completion time, when one is available. Once `status` is `fulfilled`, this is always `null` |

#### `customer`

| Field | Type | Description |
|---|---|---|
| `given_name` | string | Customer's given name. See the current response behaviour below |
| `family_name` | string | Customer's last name |
| `email` | string or null | Customer's email address |

At the moment, `customer.given_name` contains the customer's first initial rather than their full given name. You can continue to send the full given name when creating or amending a booking. If your customer journey needs the full name, keep the value you submitted and use that instead. `amendable_data.customer.given_name` also contains the first initial.

#### `product_requirements`

Customer, vehicle, flight, and party details associated with the booking. The array includes each requirement used by the booked product.

| Field | Type | Description |
|---|---|---|
| `name` | string | Requirement name, e.g. `vehicle_registration` |
| `value` | string, integer, or null | The current value held on the booking. Null means the value has not been supplied yet |
| `requirement_deadline` | UTC datetime or null | The latest moment this requirement will be reliably honoured by the supplier. Null means no specific supplier deadline is returned for that requirement |

Use `requirement_deadline` to time customer follow-ups for missing before-travel details. It is supplier guidance, not the API acceptance rule. Whether a change is accepted is governed by `policies.amendments` and the amendment quote endpoint: price-affecting fields require `policies.amendments.permitted` to be `true`, while always-amendable fields can be submitted within the `policies.amendments.until` window even when `permitted` is `false`.

Deadlines can change if the booking is amended to a different `parking_entry_datetime`. Refetch this endpoint after any successful amendment rather than caching requirement deadlines across amendments.

#### `pricing`

`pricing` contains Money objects. Use `amount_major` when displaying a price to customers. Use `amount_minor` when you need integer minor units for payment or reconciliation logic.

| Field | Type | Description |
|---|---|---|
| `total` | Money object | Total price charged |
| `commission` | Money object | Your commission on this booking |
| `commission_vat` | Money object or null | VAT on commission where applicable |

#### `policies`

| Field | Type | Description |
|---|---|---|
| `amendments.permitted` | boolean | Whether price-affecting amendments are currently permitted on this booking. Customer and vehicle details can still be amended even when this is `false`. See the [amendment quote endpoint](./patch-parking-amendment-quote.md) for which fields are price-affecting. |
| `amendments.until` | UTC datetime or null | Deadline for price-affecting amendments. We recommend updating customer and vehicle details by the same cutoff to give suppliers time to prepare on the day. |
| `refunds` | array | Refund tiers - each entry has `amount` (a Money object) and `until` (UTC datetime). Empty if non-refundable |

#### `access_methods`

The preferred field for telling customers how to access the car park. Each item has:

| Field | Type | Description |
|---|---|---|
| `priority` | integer | Display order, starting at `1`. The highest-priority method is the one to show first |
| `type` | string | One of `supplier_confirmation`, `licence_plate`, `barcode`, or `reference` |
| `value` | string or null | The value to display or render for that method. For `barcode`, this is always a stable image URL returned immediately. For `supplier_confirmation`, it is always `null` |

The response includes only the methods available for the booked product. They are returned in the order they should be shown to the customer, with priority `1` shown first. Priority values always form a continuous sequence, such as `1, 2, 3`, with no gaps.

For `barcode` methods, `value` is always returned immediately as a stable image URL and is never `null`. Render the URL as an image rather than displaying it as the customer's barrier code. Before supplier fulfilment, the URL may return `404 Not Found` with `Cache-Control: no-store`. Once the supplier confirms the booking, the same URL returns the PNG image, so you do not need to replace the value in your email, dashboard, or stored booking record. Follow the response caching headers and keep `confirmation_page_link` available as a fallback.

Some other values are only known once supplier fulfilment has completed. If a `reference` method has `value: null`, refetch the booking after `supplier_fulfilment.status` changes and wait for its value before showing that method to the customer. A `licence_plate` value remains `null` until the vehicle registration has been supplied.

See [Accessing Parking](../user-guides/accessing-parking.md#barcode-image-urls) for the full barcode implementation guidance.

`supplier_confirmation` is different: it means the supplier sends access instructions to the customer directly, usually by email. Its `value` is always `null`, so treat the supplier email as the access instruction rather than waiting for an API-delivered code.

| Type | Included when | Value |
|---|---|---|
| `supplier_confirmation` | The supplier sends the customer direct access instructions outside the Partner API | Always `null` |
| `licence_plate` | The car park uses the vehicle registration for access | The vehicle registration, or `null` if not supplied |
| `barcode` | The car park uses barcode access | Stable image URL returned immediately. The URL may return `404 Not Found` until supplier fulfilment completes, then the same URL returns the PNG image |
| `reference` | The car park uses a text reference for access | `null` until `supplier_fulfilment.status` is `fulfilled`; after that, the supplier reference, or the Holiday Extras booking reference if no supplier reference is available |

#### `product_content`

Localised product content returned in the language specified by `accept-language`. These fields mirror the response from [GET /v2/content/parking](./get-parking-content.md), except `product_code` and `reviews` are not included on booking content. See that document for the full content field reference.

#### `amendable_data`

Reflects the current values of all fields that can be submitted to the amendment endpoint. Use this to pre-populate an amendment form.

| Field | Type | Description |
|---|---|---|
| `parking_entry_datetime` | local datetime or null | Current entry time |
| `parking_exit_datetime` | local datetime or null | Current exit time |
| `customer` | object or null | Current customer details |
| `product_requirements` | object or null | Current product requirement values |

---

## Error responses

Errors follow [RFC 9457 Problem Details](../errors.md).

| Status | When |
|---|---|
| `401 Unauthorized` | Missing or expired bearer token |
| `404 Not Found` | Booking reference does not exist |
| `500 Internal Server Error` | Unexpected server error |

---

## Sandbox examples

### Happy paths

#### Get a confirmed booking

Replace `{ref}` with a `booking_reference` returned from a successful sandbox booking.

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/{ref}" \
  -H "Authorization: Bearer {token}" \
  -H "accept-language: en-GB"
```

#### Get a booking where parking time amendments are not permitted - `SBXEUNOAM01`

Returns an active booking with `policies.amendments.permitted: false`.

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/SBXEUNOAM01" \
  -H "Authorization: Bearer {token}" \
  -H "accept-language: en-GB"
```

#### Get a cancelled booking - `SBXCANC001`

Returns a booking with `booking_status: "cancelled"`.

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/SBXCANC001" \
  -H "Authorization: Bearer {token}" \
  -H "accept-language: en-GB"
```

---

### Error scenarios

#### Booking not found (404) - `SBXNOTFOUND`

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking/SBXNOTFOUND" \
  -H "Authorization: Bearer {token}" \
  -H "accept-language: en-GB"
```

---

## Related

- [POST /v2/bookings/parking](./post-parking-booking.md) - create a booking to get a reference
- [PATCH /v2/bookings/parking/{ref}/amendments/quote](./patch-parking-amendment-quote.md) - request an amendment quote for this booking
- [GET /v2/bookings/parking/{ref}/cancellations/quote](./get-parking-cancellation-quote.md) - request a cancellation quote for this booking
- [GET /v2/content/parking](./get-parking-content.md) - full reference for the product content fields
