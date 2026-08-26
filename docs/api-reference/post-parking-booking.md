# POST /v2/bookings/parking

Creates a parking booking using a `product_token` from a product search. Returns a `booking_reference` which is used for all subsequent operations on the booking.

---

## Request

```http
POST https://api.holidayextras.com/partner-api/v2/bookings/parking
```

### Headers

| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes | `Bearer {token}` - see [Authentication](../integration-guides/02-authentication.md) |
| `Content-Type` | Yes | `application/json` |
| `Idempotency-Key` | Yes | A unique key for this request, e.g. a UUID. Prevents duplicate bookings if the request is retried |

### Request body

```json
{
  "product_token": "pgtoken_a1b2c3d4",
  "partner_reference": "YOUR-REF-001",
  "customer": {
    "given_name": "Jane",
    "family_name": "Smith",
    "email": "jane.smith@example.com"
  },
  "product_requirements": {
    "vehicle_registration": "AB12 CDE",
    "outbound_flight_number": "BA123",
    "inbound_flight_number": "BA456",
    "mobile_number": "+447700900000"
  }
}
```

### Request fields

**Required**

| Field | Type | Description |
|---|---|---|
| `product_token` | string | Token from the product search response. Use the token for the product and quoted price the customer selected |
| `customer.given_name` | string | Customer's first name |
| `customer.family_name` | string | Customer's last name |

**Optional**

| Field | Type | Description |
|---|---|---|
| `partner_reference` | string | Your own reference for this booking |
| `customer.email` | string | Customer's email address |

**`product_requirements`**

Which fields to include here varies per product. The search response includes a `product_requirements` array on each product result - each entry has a `required_at` value that tells you when the field needs to be collected:

- **`"at_booking"`** - include in this request. The booking won't complete without it. Vehicle registration is usually in this category.
- **`"before_travel"`** - needed before the customer travels, but the booking completes without it. Fields like vehicle make, model, and colour are often in this category. These can be collected post-purchase and submitted via amendment, which keeps checkout friction low.

Only include fields listed in the product's `product_requirements` array. If you pass a field the product has not declared, the API returns a validation error so you can remove that field and try again.

| Field | Type | Description |
|---|---|---|
| `vehicle_registration` | string | Vehicle registration. Some car parks use it for access or to identify the vehicle |
| `outbound_flight_number` | string | Outbound flight number, e.g. `BA123` |
| `inbound_flight_number` | string | Inbound flight number, e.g. `BA456` |
| `mobile_number` | string | Customer's mobile number |
| `vehicle_make` | string | Vehicle make, e.g. `Toyota` |
| `vehicle_model` | string | Vehicle model, e.g. `Corolla` |
| `vehicle_colour` | string | Vehicle colour, e.g. `Silver` |
| `destination` | string | The customer's destination |

---

## Response

### 201 Created

```json
{
  "booking_reference": "GSFWRJ",
  "partner_reference": "YOUR-REF-001",
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
  }
}
```

### Response fields

| Field | Type | Description |
|---|---|---|
| `booking_reference` | string | The Holiday Extras booking reference. Use this for all subsequent operations |
| `partner_reference` | string | Your reference, echoed back |
| `pricing.total` | Money object | Total price charged |
| `pricing.commission` | Money object | Your commission on this booking |
| `pricing.commission_vat` | Money object or null | VAT on commission where applicable |

Money objects contain `amount_major`, `amount_minor`, and `currency`. Use `amount_major` for display and `amount_minor` for integer minor-unit handling.

---

## Error responses

Errors follow [RFC 9457 Problem Details](../errors.md).

| Status | When |
|---|---|
| `400 Bad Request` | Missing or invalid request body fields |
| `401 Unauthorized` | Missing or expired bearer token |
| `409 Conflict` | The quoted price can no longer be accepted. See the price changed response below |
| `409 Conflict` | A booking with this idempotency key already exists with a different request body |
| `422 Unprocessable Entity` | Product token has expired - re-run the search |
| `422 Unprocessable Entity` | Product is no longer available |
| `500 Internal Server Error` | Unexpected server error |

### Price changed response

When the quoted price can no longer be accepted, the API returns `409 Conflict` using the standard problem-details shape before a booking is created. See [Price Lock](../integration-guides/07-price-lock.md) for the partner checkout options when a replacement is available. The response may also include:

| Field | Type | Description |
|---|---|---|
| `validated_price` | Money object or null | The latest price returned for the selected product, when a replacement token can also be issued |
| `alternative_product_token` | string or null | A fresh token for the same product at the latest price, when the product is still available |

Continue with the alternative product only when both `validated_price` and `alternative_product_token` are present. You can show the customer the updated price or absorb the difference yourself. If you show the updated price, ask the customer to accept it before continuing. Submit a new booking request with the alternative token and a new `Idempotency-Key`. If either field is missing, run a fresh search before showing a new price to the customer.

```json
{
  "type": "https://docs.holidayextras.co.uk/partner/v2/problems/conflict",
  "title": "Conflict",
  "status": 409,
  "code": "conflict",
  "trace_id": "trace_123",
  "errors": [
    {
      "message": "The quote is older than the Price Lock window."
    }
  ],
  "validated_price": {
    "amount_major": 94.99,
    "amount_minor": 9499,
    "currency": "GBP"
  },
  "alternative_product_token": "pgtoken_fresh_quote"
}
```

---

## Sandbox examples

### Happy paths

#### Successful booking - `LTN` / `GBP`

Run a search first to get a `product_token`, then use it here. The sandbox returns a stable booking reference.

```bash
curl -X POST "https://api-sandbox.holidayextras.com/partner-api/v2/bookings/parking" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: $(uuidgen)" \
  -d '{
    "product_token": "{token_from_search}",
    "partner_reference": "YOUR-REF-001",
    "customer": {
      "given_name": "Jane",
      "family_name": "Smith",
      "email": "jane.smith@example.com"
    },
    "product_requirements": {
      "vehicle_registration": "AB12 CDE"
    }
  }'
```

#### Replay - same idempotency key

Resending a request with the same `Idempotency-Key` returns the original response without creating a duplicate booking.

---

### Error scenarios

#### Expired product token (422)

Wait for `product_token_valid_until` to pass before submitting, or pass a manually expired token.

#### Price changed since search (409)

Triggered in the sandbox when the quoted price can no longer be accepted. Continue only when both `validated_price` and `alternative_product_token` are returned. You can show the customer the updated price and ask them to accept it, or absorb the difference yourself. Submit a new booking request with the alternative token and a new `Idempotency-Key`. If either field is missing, re-run the search before showing a new price.

---

## Related

- [GET /v2/products/parking/detailed](./get-parking-availability-detailed.md) - get a `product_token` to use in this request
- [GET /v2/bookings/parking/{ref}](./get-parking-booking.md) - retrieve the booking after creation
- [Selling Parking](../user-guides/selling-parking.md) - what to collect from customers at booking
