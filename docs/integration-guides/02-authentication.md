# Authentication

The API uses OAuth 2.0 `client_credentials` flow - a straightforward way to get a token that covers all your requests.

## Get a Token

```http
POST https://auth-staging.holidayextras.com/oauth2/token
Authorization: Basic {base64(client_id:client_secret)}
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
```

### Response

```json
{
  "access_token": "eyJhbGci...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

## Use the Token

Include the token as a Bearer header and it takes care of authentication for all requests:

```http
Authorization: Bearer eyJhbGci...
```

## Token Caching

Tokens are valid for `expires_in` seconds (3600 = 1 hour). Caching and reusing the token keeps things efficient - one fetch per hour rather than one per request. A 60-second buffer before expiry ensures a fresh token is always ready:

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

## Sandbox Credentials

To get your sandbox `client_id` and `client_secret`, contact [partnerconnect@holidayextras.com](mailto:partnerconnect@holidayextras.com). These are scoped to your account. Keep your `client_secret` secure.

---

Next: [Sandbox testing](./03-sandbox-testing.md)
