# Bad Request

| Field | Value |
|-------|-------|
| **Status** | 400 |
| **Code** | `bad_request` |

The request was malformed or missing required parameters. Check the `errors` array for details on which fields failed validation.

## Common causes

- Missing required query parameters (e.g. `location_code`, `currency`)
- Invalid parameter format (e.g. non-ISO datetime, invalid IATA code)
- Missing or malformed request body on POST/PATCH endpoints
- An HTTP method and path combination the API does not support. `detail` names it, e.g. `Operation PATCH /v2/locations is not supported`

## Example response

```json
{
  "type": "https://github.com/holidayextras/partner-api-documentation/tree/main/docs/problems/bad-request",
  "title": "Bad Request",
  "status": 400,
  "code": "bad_request",
  "errors": [
    {
      "field": "location_code",
      "message": "must match pattern \"^[A-Z]{3}$\""
    }
  ]
}
```

## How to resolve

Review the `errors` array to identify which fields need correcting. Each error includes a `field` name and a `message` describing the validation failure.

> **Note:** The `errors` array is always present in every error response. It will contain field-level details when applicable, or be empty (e.g., for 500 errors).
