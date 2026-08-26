# Version migrations

This section holds guides for moving your integration from one major version of the API to the next. Each guide explains what changed and the steps to update.

The API is versioned by major in the path (`/v1`, `/v2`, …). Within a major, changes are only ever additive and backwards-compatible - breaking changes arrive as a new major. When we introduce a new major, the previous one keeps working for a transition period, so you can migrate on your own schedule rather than all at once.

---

## How you'll know it's time to migrate

You don't need to poll for this. A version that's on its way out keeps serving your requests exactly as before, but adds headers to every response telling you it's deprecated and where to go:

```http
Deprecation: @1767225600
Sunset: Thu, 15 Jul 2027 00:00:00 GMT
Link: <https://github.com/holidayextras/partner-api-connect-documentation/blob/staging/docs/migration-guides/v2-to-v3.md>; rel="sunset"
```

- **`Deprecation`** - this version is deprecated; start planning your move.
- **`Sunset`** - the date after which it will stop serving requests.
- **`Link`** (`rel="sunset"`) - the migration guide for this version, published in this section.

The `API-Version` header on every response tells you which major you're currently talking to.

---

## After a version is retired

Once a version has been sunset, requests to it return **`410 Gone`** with a link to its migration guide instead of a normal response. Migrating before the sunset date avoids any interruption.

---

## Guides

Each guide is named for the transition it covers - for example `v2-to-v3.md`.

There are no migration guides yet: the API currently runs a single major version, with nothing deprecated. Guides will appear here as new majors are introduced.
