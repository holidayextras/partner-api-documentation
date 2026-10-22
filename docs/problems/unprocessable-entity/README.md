# Unprocessable Entity

| Field | Value |
|-------|-------|
| **Status** | 422 |
| **Code** | `unprocessable_entity` |

The request is well-formed but contains values that cannot be processed. Check the `errors` array for details.

## Common causes

- Product token has expired or is invalid
- The booking is not in a state that allows the request (e.g. it has no car park product)

## Example response

```json
{
  "type": "https://github.com/holidayextras/partner-api-documentation/tree/main/docs/problems/unprocessable-entity",
  "title": "Unprocessable Entity",
  "status": 422,
  "code": "unprocessable_entity",
  "errors": [
    {
      "field": "product_token",
      "message": "Product token has expired"
    }
  ]
}
```

## How to resolve

Review the `errors` array for specific field-level details. Unlike a 400 (malformed request), a 422 means the request format is correct but the values are not valid for processing.
