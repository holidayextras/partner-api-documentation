# Search endpoints

Product search can follow one of two patterns. A detailed endpoint returns availability, pricing, and localised product content together. Separate search and content endpoints let you fetch current availability and pricing while caching content independently.

---

## Detailed search

Use detailed search when you want pricing, availability, and content in a single response. It is a straightforward starting point because there are no additional content calls or caching layer to manage.

The exact endpoint path and content fields depend on the product.

---

## Separate search and content

Use separate endpoints when you want to cache product content and request only current availability and pricing during each search.

Product content such as names, images, descriptions, directions, and facilities changes less often than price and availability. Fetching it separately means you can cache and reuse it across searches.

A typical pattern looks like this:

1. Fetch and cache content for the products and locations you support
2. Call the product search endpoint for current pricing and availability during each customer search
3. Merge the cached content with the fresh pricing to build the product listing

Pricing and availability should be fetched fresh for each search. Content can be refreshed on a schedule that fits your integration and the product.

---

## Which to use

| | Detailed endpoint | Separate endpoints |
|---|---|---|
| Setup complexity | Lower - single request | Higher - requires a caching layer |
| Response size per search | Larger | Smaller |
| Content freshness | Always fresh | Cached - refresh periodically |
| Best for | Most integrations | High-traffic integrations where search performance matters |

---

## Airport Parking endpoints

For Airport Parking:

- `GET /v2/products/parking/detailed` returns availability, pricing, and localised content together.
- `GET /v2/products/parking` returns availability and pricing without content.
- `GET /v2/content/parking` returns content for the requested product codes.

Both parking search endpoints return `price_lock_valid_until` for every result. This gives you the Price Lock window for the selected price if the customer goes on to book.

See [Search parking with content](../api-reference/get-parking-availability-detailed.md), [Search parking](../api-reference/get-parking-availability.md), [Parking content](../api-reference/get-parking-content.md), and [Price Lock](./07-price-lock.md).

---

Next: [Price Lock](./07-price-lock.md)
