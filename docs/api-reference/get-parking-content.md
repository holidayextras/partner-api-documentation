# GET /v2/content/parking

Returns localised product content for one or more parking products.

> **Review data availability:** Parking review fields are included in each content object. The rating value, review count, and summary are nullable, so they return `null` when review data is not available for a car park. The rating scale metadata is always present.

## Using parking content

The response includes content for different points in the customer journey. Use the fields that help customers compare products, check that a product suits their needs, or prepare for travel.

Most fields are nullable and vary by product. Show content when it has a value and is relevant to the parking service. For recommendations on presenting content before booking, see [Selling parking](../user-guides/selling-parking.md). For confirmation and day-of-travel information, see [Accessing parking](../user-guides/accessing-parking.md).

---

## Request

```http
GET https://api.holidayextras.com/partner-api/v2/content/parking
```

### Headers

| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes | `Bearer {token}` - see [Authentication](../integration-guides/02-authentication.md) |
| `accept-language` | Yes | Language for content fields. One of: `en-GB`, `de-DE`, `nl-NL`, `fr-FR`, `es-ES`, `it-IT`, `pt-PT`, `pl-PL`, `cy-GB` |

### Query parameters

**Required**

| Parameter | Type | Description |
|---|---|---|
| `product_codes` | array | One or more product codes from the availability search. Up to 50 per request |

### Example request

```bash
curl "https://api.holidayextras.com/partner-api/v2/content/parking\
?product_codes=LPH4\
&product_codes=NCT0\
&product_codes=BHW8" \
  -H "Authorization: Bearer {token}" \
  -H "accept-language: en-GB"
```

---

## Response

### 200 OK

Returns an array of product content objects. Products with no available content are omitted from the response.

```json
[
  {
    "product_code": "LPH4",
    "site_name": "Long Stay South Gatwick",
    "language": "en-GB",
    "name": "Long Stay South",
    "terminal": "South",
    "parking_type": "park_and_ride",
    "selling_point": "Great value long stay parking at Gatwick",
    "address": "Longbridge Way, Crawley RH6 0NX",
    "telephone": "+44 1293 000000",
    "longitude": -0.1821,
    "latitude": 51.1537,
    "distance_miles": 1.2,
    "transfer_frequency_minutes": 10,
    "transfer_duration_minutes": 8,
    "operating_time_earliest": "04:00",
    "operating_time_latest": "23:00",
    "transfer_overview": "Shuttle buses run every 10 minutes and take 8 minutes to reach the terminal.",
    "is_vehicle_parked_for_you": false,
    "is_electric_charging_available": true,
    "is_electric_charging_included": false,
    "is_under_cover": false,
    "licence_plate_access": true,
    "bar_code_access": false,
    "directions": "Follow signs for Long Stay South from the M23...",
    "arrival_procedures": "Drive to the barrier and your plate will be recognised automatically...",
    "departure_procedures": "Take the shuttle from stand 3 on your return...",
    "desktop_image": "https://cdn.example.com/parking/lgw-lph4-desktop.jpg",
    "mobile_image": "https://cdn.example.com/parking/lgw-lph4-mobile.jpg",
    "wallet_image": "https://cdn.example.com/parking/lgw-lph4-wallet.jpg",
    "reviews": {
      "rating": {
        "value": null,
        "scale_min": 1,
        "scale_max": 10
      },
      "count": null,
      "summary": null
    },
    "..." : "..."
  }
]
```

### Response fields

Every content object includes `product_code`, `language`, `parking_type`, and a `reviews` object. Other content fields are nullable. Review values inside `reviews` may be `null` when review data is not available for a car park.

#### Identity and location

| Field | Type | What it contains | Typical use |
|---|---|---|---|
| `product_code` | string | The product code this content belongs to | Match the content to a product returned by an availability search. This is for integration logic rather than customer display |
| `site_name` | string or null | Name of the car park site. Products sharing a `site_name` are variants of the same location | Group product variants in listings and use as the shared customer-facing site name |
| `language` | string | Language of the content fields, matching the `accept-language` request header | Cache and serve the correct translation. This does not normally need to be displayed |
| `name` | string or null | Product display name | Product listings, detail views, booking confirmations and travel reminders |
| `terminal` | string or null | Airport terminal, if applicable | Listings and detail views where terminal coverage helps customers compare products |
| `address` | string or null | Full postal address | Product details, booking confirmation and day-of-travel directions |
| `telephone` | string or null | Car park or operator contact number | Booking confirmation, travel reminders and day-of-travel help |
| `what3words` | string or null | what3words location reference | Booking confirmation or travel directions where a precise location is helpful |
| `longitude` | number or null | WGS84 longitude | Maps, route planning and other location-based integration features |
| `latitude` | number or null | WGS84 latitude | Maps, route planning and other location-based integration features |
| `distance_miles` | number or null | Distance from the airport terminal | Product listings, comparison and detail views |

#### Parking type

| Field | Type | What it contains | Typical use |
|---|---|---|---|
| `parking_type` | string | One of `meet_and_greet`, `return_greet`, `park_and_stroll`, `park_and_ride`, `economy_parking` | Label and filter products, and explain the service customers are choosing |
| `is_vehicle_parked_for_you` | boolean or null | Whether staff move the vehicle on behalf of the customer | Product comparison and detail views where vehicle handling may affect the customer's choice |
| `licence_plate_access` | boolean or null | Whether the product supports licence plate recognition | Explain the product's usual access method before booking. For a specific booking, use `access_methods` from the booking response |
| `bar_code_access` | boolean or null | Whether the product supports barcode access | Explain the product's usual access method before booking. For a specific booking, use `access_methods` from the booking response |

#### Transfer

| Field | Type | What it contains | Typical use |
|---|---|---|---|
| `transfer_overview` | string or null | Prose summary of the transfer provision and its constraints, such as how often the shuttle runs, its operating hours, and any luggage or child seat arrangements | Product details and travel information. Prefer this over assembling a summary from the individual transfer fields |
| `transfer_frequency_minutes` | number or null | How often the shuttle runs | Product listings and comparison for products with a shuttle transfer |
| `transfer_duration_minutes` | number or null | Journey time to the terminal | Product listings, comparison and detail views for products with a shuttle transfer |
| `transfer_maximum_passengers` | number or null | Maximum passengers per vehicle | Product details before booking, particularly for larger groups |
| `transfer_extra_passengers_price` | number or null | Additional cost for extra passengers, in major units of the product's currency | Product details before booking when the customer's party may incur an extra charge |

#### Facilities and access

| Field | Type | What it contains | Typical use |
|---|---|---|---|
| `maximum_vehicle_size` | string or null | Height or size restriction | Product details and suitability checks before booking |
| `operating_time_earliest` | string or null | Earliest time the product's service operates | Product details and travel information when operating hours may affect the customer's trip |
| `operating_time_latest` | string or null | Latest time the product's service operates | Product details and travel information when operating hours may affect the customer's trip |
| `is_under_cover` | boolean or null | Whether the parking is under cover | Filters, listings and detail views where covered parking is a differentiator |
| `is_electric_charging_available` | boolean or null | Whether EV charging is available | Filters, listings and detail views for customers looking for EV charging |
| `is_electric_charging_included` | boolean or null | Whether EV charging is included in the price | Product details alongside EV charging availability, so customers know whether it is included |

#### Procedures

| Field | Type | What it contains | Typical use |
|---|---|---|---|
| `directions` | string or null | Directions to the car park | Product details where useful, then booking confirmation and travel reminders |
| `arrival_procedures` | string or null | What to do when arriving at the car park | Product details where the process helps explain the service, then booking confirmation and travel reminders |
| `departure_procedures` | string or null | What to do when returning from the airport | Product details where the process helps explain the service, then booking confirmation and travel reminders |

#### Marketing and display

| Field | Type | What it contains | Typical use |
|---|---|---|---|
| `selling_point` | string or null | Short marketing headline | Product listings and the top of a product detail view |
| `information` | string or null | General product information | Product details and post-booking information where it helps the customer prepare |
| `info_tab` | string or null | Additional product information, including on-site security and accessible facilities | Product detail views |
| `desktop_image` | string or null | Image URL optimised for desktop | Desktop product listings or detail views |
| `mobile_image` | string or null | Image URL optimised for mobile | Mobile product listings or detail views |
| `wallet_image` | string or null | Image URL for wallet or pass display | Compact booking confirmations, account areas and wallet passes |
| `image_gallery` | array of strings or null | Complete gallery of product image URLs | Product detail views where customers can explore the car park before booking |

#### Reviews

These fields provide site-level review information for the car park. Products that are variants of the same site may share the same review information. The rating value, count, and summary return `null` when review data is not available.

| Field | Type | What it contains | Typical use |
|---|---|---|---|
| `rating.value` | number or null | Average customer rating. Will return `null` when review data is not available | Product listings and detail views, shown with the rating scale and review count |
| `rating.scale_min` | integer | Minimum value on the rating scale. For parking reviews this is `1` | Format and label the rating correctly. This does not normally need to be displayed on its own |
| `rating.scale_max` | integer | Maximum value on the rating scale. For parking reviews this is `10` | Format and label the rating correctly, for example `8.9/10` |
| `count` | integer or null | Number of reviews included in the rating. Will return `null` when review data is not available | Product listings and detail views alongside the average rating |
| `summary` | string or null | Short review summary. Will return `null` when a summary is not available | Product listings or detail views where a short summary helps customers compare options |

---

## Error responses

Errors follow [RFC 9457 Problem Details](../errors.md).

| Status | When |
|---|---|
| `400 Bad Request` | Missing or invalid query parameters |
| `401 Unauthorized` | Missing or expired bearer token |
| `406 Not Acceptable` | Missing or unsupported `accept-language` value |
| `500 Internal Server Error` | Unexpected server error |

---

## Sandbox examples

### Happy paths

#### English content - `en-GB`

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/content/parking\
?product_codes=LPH4\
&product_codes=NCT0" \
  -H "Authorization: Bearer {token}" \
  -H "accept-language: en-GB"
```

#### German content - `de-DE`

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/content/parking\
?product_codes=FMMJ\
&product_codes=BER7" \
  -H "Authorization: Bearer {token}" \
  -H "accept-language: de-DE"
```

---

### Validation errors

#### Missing accept-language header

Triggers a `406` response.

```bash
curl "https://api-sandbox.holidayextras.com/partner-api/v2/content/parking\
?product_codes=LPH4" \
  -H "Authorization: Bearer {token}"
```

---

## Related

- [Search endpoints](../integration-guides/04-search-endpoints.md) - when to use this endpoint vs the detailed endpoint
- [GET /v2/products/parking](./get-parking-availability.md) - search for available products, returning `code` values to pass here
- [GET /v2/products/parking/detailed](./get-parking-availability-detailed.md) - search with content included in a single response
- [Selling Parking](../user-guides/selling-parking.md) - which content fields to surface and how
