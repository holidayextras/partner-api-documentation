# POST /oauth2/token

Exchanges your client credentials for a Bearer token. The token is valid for 1 hour and covers all API requests during that window - caching and reusing it keeps your integration efficient.

---

## Request

```http
POST https://auth.holidayextras.com/oauth2/token
```

### Headers

| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes | `Basic {base64(client_id:client_secret)}` - your credentials encoded as described below |
| `Content-Type` | Yes | `application/x-www-form-urlencoded` |

### Building the Authorization header

Concatenate your `client_id` and `client_secret` with a colon, then base64-encode the result:

```
base64("your_client_id:your_client_secret")
```

### Request body

| Field | Value |
|---|---|
| `grant_type` | `client_credentials` |

### Example request

```bash
curl -X POST "https://auth.holidayextras.com/oauth2/token" \
  -H "Authorization: Basic $(echo -n 'your_client_id:your_client_secret' | base64)" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials"
```

---

## Response

### 200 OK

```json
{
  "access_token": "eyJhbGci...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

### Response fields

| Field | Type | Description |
|---|---|---|
| `access_token` | string | The Bearer token to include on all API requests |
| `token_type` | string | Always `Bearer` |
| `expires_in` | integer | Seconds until the token expires. Typically `3600` (1 hour) |

---

## Using the token

Include the token as a Bearer header on every API request:

```http
Authorization: Bearer eyJhbGci...
```

### Caching

A token is valid for `expires_in` seconds. Caching it and refreshing shortly before expiry means one token fetch per hour rather than one per request. A 60-second buffer before `expires_in` ensures a fresh token is always ready:

```js
let tokenCache = { token: null, expiresAt: 0 };

async function getToken() {
  if (tokenCache.token && Date.now() < tokenCache.expiresAt) {
    return tokenCache.token;
  }
  const data = await fetchNewToken();
  tokenCache.token = data.access_token;
  tokenCache.expiresAt = Date.now() + (data.expires_in - 60) * 1000;
  return tokenCache.token;
}
```

---

## Error responses

| Status | When |
|---|---|
| `401 Unauthorized` | Invalid `client_id` or `client_secret` |
| `400 Bad Request` | Missing or invalid `grant_type` |

---

## Sandbox example

The sandbox uses a separate auth host:

```bash
curl -X POST "https://auth-staging.holidayextras.com/oauth2/token" \
  -H "Authorization: Basic $(echo -n 'your_client_id:your_client_secret' | base64)" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials"
```

---

## Related

- [Authentication](../integration-guides/02-authentication.md) - full guide on token management and caching
- [API Overview](../integration-guides/01-api-overview.md) - how authentication fits into the integration journey
