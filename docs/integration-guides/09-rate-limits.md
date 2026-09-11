# Rate limits

Partner API applies a request rate limit to every endpoint. Limits protect the platform and our suppliers from traffic spikes, and keep response times predictable for all partners.

Limits are applied per set of credentials. Credentials are issued per environment, so your sandbox, staging, and production traffic each have their own limit. The same defaults apply in every environment.

---

## Default limits

| Endpoint group | Endpoints | Default limit |
|---|---|---|
| Read | `GET /v2/locations`, `GET /v2/products/parking`, `GET /v2/products/parking/detailed`, `GET /v2/content/parking` | 5 requests per second |
| Booking | `POST /v2/bookings/parking`, `GET /v2/bookings/parking/{ref}`, `PATCH /v2/bookings/parking/{ref}/amendments/quote`, `POST /v2/bookings/parking/{ref}/amendments/confirm`, `GET /v2/bookings/parking/{ref}/cancellations/quote`, `POST /v2/bookings/parking/{ref}/cancellations/confirm` | 1 request per second |

Both quote endpoints and `GET /v2/bookings/parking/{ref}` sit in the booking group, even though they are read requests. They reach the booking platform and its suppliers, so they carry the lower limit.

`GET /v2/schema.json` and CORS preflight `OPTIONS` requests are not rate limited.

Limits are per second rather than per day or per month. There is no monthly request quota.

If your expected volumes are higher than these defaults, raise it with your Holiday Extras partnerships contact during onboarding, with the endpoints and the peak rate you expect.

---

## How the limit is enforced

The limit is a smoothed rate, not a burst allowance. A 5 requests per second limit is enforced as roughly one request every 200 milliseconds, and a 1 request per second limit as roughly one request every second. Five search requests sent at the same instant can therefore return `429`, even though the total for that second is within the limit.

Spread requests evenly rather than sending them in bursts. In practice this means:

- Space concurrent searches rather than fanning out one request per location or per date at once.
- Queue booking, amendment, and cancellation writes so that only one is in flight at a time.
- Keep a small client-side delay between retries rather than retrying immediately.

---

## Response headers

Every rate-limited response includes the current limit:

```http
RateLimit-Limit: 5
RateLimit-Remaining: 5
RateLimit-Reset: 1780547930
```

| Header | Description |
|---|---|
| `RateLimit-Limit` | Requests allowed in the current window for the endpoint group |
| `RateLimit-Remaining` | Requests still allowed in the current window |
| `RateLimit-Reset` | Unix timestamp, in seconds, when the current window resets |

Because the limit is smoothed rather than counted as a bucket, `RateLimit-Remaining` reports the configured allowance rather than a live countdown. Use it to read your current limit, not to predict when the next request will be rejected.

`429` responses also include `Retry-After`:

```http
Retry-After: 1
```

`Retry-After` is the number of seconds to wait before sending the next request.

---

## The 429 response

When the limit is exceeded, the API returns `429 Too Many Requests` with an RFC 7807 Problem Details body:

```json
{
  "type": "https://docs.holidayextras.co.uk/partner/v2/problems/too-many-requests",
  "title": "Too Many Requests",
  "status": 429,
  "code": "too_many_requests",
  "detail": "Request rate limit exceeded. See the Retry-After header for wait time.",
  "errors": []
}
```

A `429` means the request never reached the booking platform. Nothing was created, amended, or cancelled, so the request is always safe to retry.

---

## Handling 429 in your integration

1. Read `Retry-After` and wait at least that long.
2. Retry with exponential backoff and jitter if the retry is also rejected, for example 1s, 2s, 4s, 8s with a small random offset. Jitter stops parallel workers from retrying in lockstep.
3. Cap the number of retries, then surface a recoverable error rather than retrying indefinitely.
4. Retry writes with the same `Idempotency-Key`. A `429` did not create anything, and reusing the key keeps the retry safe if an earlier attempt did get through.
5. Keep the customer informed on customer-facing screens while a retry is in progress, and avoid letting them resubmit the same action.
6. Log `429` responses with the endpoint and timestamp. A rising rate is a signal to review your request patterns or ask for a higher limit before it affects customers.

Do not treat `429` as a failed booking. Retrying after the wait period is the expected behaviour.

Sandbox applies the same limits and returns fixed responses without reaching suppliers, so it is a safe place to exercise your `429` handling.

---

## Auth server limits

The token endpoint is rate limited separately, per `client_id`:

| Limit | Value | Applies to |
|---|---|---|
| Request rate | 10 requests per second | `POST /oauth2/token`, `POST /oauth2/revoke` |
| Hourly quota | 100 requests per hour | `POST /oauth2/token`, `POST /oauth2/revoke` |

The hourly quota is the one to design around. Tokens are valid for one hour, so an integration that caches its token uses roughly one request per hour. An integration that fetches a fresh token for every API call exhausts the quota after 100 calls and cannot authenticate again until the hour rolls over. See [Token caching](./02-authentication.md#token-caching).

Auth server errors do not use Problem Details. When either limit is exceeded, the response is `429` with:

```json
{
  "error": {
    "message": "Too Many Requests"
  }
}
```

The auth server returns its own rate limit headers on every response. These describe the hourly quota, and unlike the Partner API headers above, `RateLimit-Remaining` is a live countdown, so you can see how much of your hour is left before you run out:

```http
RateLimit-Limit: 100
RateLimit-Remaining: 87
RateLimit-Reset: 1780551530
```

| Header | Description |
|---|---|
| `RateLimit-Limit` | Requests allowed in the current hour |
| `RateLimit-Remaining` | Requests still allowed in the current hour |
| `RateLimit-Reset` | Unix timestamp, in seconds, when the hourly quota resets |

A `429` adds `Retry-After` and reports whichever limit you exceeded, so you can tell the two apart:

| Limit exceeded | `RateLimit-Limit` | `Retry-After` |
|---|---|---|
| 10 requests per second | `10` | `1` |
| 100 requests per hour | `100` | Seconds remaining until the quota resets |

Read `Retry-After` and wait at least that long. A `Retry-After` of `1` is a short burst you can safely retry through. A wait of minutes means the hourly quota is exhausted, and retrying sooner will keep failing until it resets — treat that as a token caching problem rather than something to retry through. See [Token caching](./02-authentication.md#token-caching).

---

## Reducing the requests you send

Most `429` responses come from request patterns that can be avoided:

- **Cache locations.** `GET /v2/locations` is stable. A daily refresh is enough.
- **Cache product content by product code and language.** Content changes far less often than price and availability.
- **Search once per customer journey.** Search for the customer's actual location, dates, and currency rather than pre-fetching every combination.
- **Use webhooks instead of polling.** Subscribe to `booking.impacted` and refetch the affected booking, rather than polling `GET /v2/bookings/parking/{ref}` on a schedule.
- **Deduplicate retries.** Make sure a retry loop in your integration cannot run at the same time as a customer-triggered request for the same action.

See [API overview](./01-api-overview.md) for the caching patterns in full, and [Webhooks](./05-webhooks.md) for booking updates without polling.

---

## Related guides

- [Error handling](../errors.md)
- [API overview](./01-api-overview.md)
- [Webhooks](./05-webhooks.md)
- [Sandbox testing](./03-sandbox-testing.md)

---

Next: [API reference](../../README.md#api-reference)
