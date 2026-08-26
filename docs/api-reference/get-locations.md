# GET /v2/locations

Returns the list of locations available for a given product type. Use this endpoint to populate an airport or location picker and to validate that a location is supported before running a product search.

---

## Request

```http
GET https://api.holidayextras.com/partner-api/v2/locations
```

### Headers

| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes | `Bearer {token}` - see [Authentication](../integration-guides/02-authentication.md) |

### Query parameters

**Required**

| Parameter | Type | Description |
|---|---|---|
| `product_types` | array | Product types to filter locations by. One or more of: `parking`, `hotels`, `hotelsWithParking`, `lounges`, `fastTrack` |

**Optional**

| Parameter | Type | Description |
|---|---|---|
| `country_codes` | array | Filter results to specific countries. Two-letter ISO 3166-1 alpha-2 codes, e.g. `GB`, `DE` |

### Example request

```bash
curl "https://api.holidayextras.com/partner-api/v2/locations?product_types=parking" \
  -H "Authorization: Bearer {token}"
```

Filtered to a specific country:

```bash
curl "https://api.holidayextras.com/partner-api/v2/locations?product_types=parking&country_codes=GB" \
  -H "Authorization: Bearer {token}"
```

### Passing more than one value

A comma-separated list is not accepted and returns a `400`.

```bash
curl "https://api.holidayextras.com/partner-api/v2/locations\
?product_types=parking\
&product_types=lounges\
&country_codes=GB\
&country_codes=DE" \
  -H "Authorization: Bearer {token}"
```

---

## Response

### 200 OK

Returns an array of location objects. Each item is a location grouped by country.

```json
[
  {
    "country_code": "GB",
    "location_name": "London Gatwick Airport",
    "supported_codes": [
      {
        "location_type": "iata",
        "location_code": "LGW"
      }
    ],
    "supported_product_types": ["parking"],
    "supported_currencies": ["GBP"]
  },
  {
    "country_code": "GB",
    "location_name": "London Heathrow Airport",
    "supported_codes": [
      {
        "location_type": "iata",
        "location_code": "LHR"
      }
    ],
    "supported_product_types": ["parking"],
    "supported_currencies": ["GBP"]
  },
  {
    "country_code": "DE",
    "location_name": "Frankfurt Airport",
    "supported_codes": [
      {
        "location_type": "iata",
        "location_code": "FRA"
      }
    ],
    "supported_product_types": ["parking"],
    "supported_currencies": ["EUR"]
  }
]
```

### Response fields

| Field | Type | Description |
|---|---|---|
| `country_code` | string | ISO 3166-1 alpha-2 country code, e.g. `GB`, `DE` |
| `location_name` | string | Display name of the airport or location in the English language |
| `supported_codes` | array | Location identifiers for this location - see below |
| `supported_product_types` | array of strings | Product types available at this location |
| `supported_currencies` | array of strings | ISO 4217 currency codes accepted for product searches at this location, e.g. `["GBP"]`, `["EUR"]` |

#### `supported_codes`

| Field | Type | Description |
|---|---|---|
| `location_type` | string | The type of location identifier. Currently always `iata` |
| `location_code` | string | Three-letter IATA airport code, e.g. `LGW` |

---

## Usage notes

### Building a location picker

Use `location_name` as the display label. The `location_code` from within `supported_codes` is the value to pass into subsequent product search requests.

### Caching

This list changes infrequently. You can cache the response and refresh it periodically rather than fetching it on every page load. A daily refresh is sufficient for most integrations, but check the `Cache-Control` header in the response.

### Passing the code to product search

The `location_code` from `supported_codes` maps directly to the `location_code` query parameter on the product search endpoint:

```
GET /v2/products/parking/detailed?location_type=iata&location_code=LGW&...
```

### Choosing a currency

Use `supported_currencies` to determine which currencies are available for the selected location. The API currently accepts `GBP` and `EUR`. If you search using one of these currencies at a location that does not support it, the response is an empty array. Values outside the API's accepted currencies return a validation error.

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

#### Parking locations

Returns all locations that support parking products.

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/locations?product_types=parking" \
  -H "Authorization: Bearer {token}"
```

#### Filter by country

Returns parking locations in the UK only.

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/locations?product_types=parking&country_codes=GB" \
  -H "Authorization: Bearer {token}"
```

---

## Related

- [API Overview](../integration-guides/01-api-overview.md) - how the locations endpoint fits into the full integration journey
- [GET /v2/products/parking/detailed](./get-parking-availability-detailed.md) - search for available parking products at a location
- [GET /v2/products/parking](./get-parking-availability.md) - search for available parking products, pricing only
