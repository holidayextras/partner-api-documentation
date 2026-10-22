# Resource Not Found

| Field | Value |
|-------|-------|
| **Status** | 404 |
| **Code** | `resource_not_found` |

The requested resource does not exist. This is a permanent error -- retrying with the same parameters will not succeed.

## Common causes

- Booking reference does not exist
- The booking reference was mistyped, or belongs to a different environment

## Example response

```json
{
  "type": "https://github.com/holidayextras/partner-api-documentation/tree/main/docs/problems/resource-not-found",
  "title": "Resource not found",
  "status": 404,
  "code": "resource_not_found",
  "errors": [
    {
      "field": "reference",
      "message": "Booking with reference ZZZZZZQ not found"
    }
  ]
}
```

## How to resolve

Verify the identifier you are using (e.g. booking reference) is correct. A 404 means the resource does not exist -- do not retry automatically.
