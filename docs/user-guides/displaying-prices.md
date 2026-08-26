# Displaying prices

Prices in the API are returned as Money objects. Each Money object includes `amount_major`, `amount_minor`, and `currency`. The API currently supports `GBP` and `EUR`.

This is the currency used for the Holiday Extras booking and partner invoicing. If you take payment from the customer in another currency, you can convert and display the customer price in your own checkout flow.

Use `amount_major` when displaying a price to customers. Use `amount_minor` when you need integer minor units for payment or reconciliation logic. Once you have the amount and currency you want to display, formatting it with the right symbol, decimal places, and locale is straightforward using the browser's built-in `Intl.NumberFormat` API.

---

## The basics

```js
function formatPrice(money) {
  return new Intl.NumberFormat(undefined, {
    style: 'currency',
    currency: money.currency,
  }).format(money.amount_major);
}

formatPrice({ amount_major: 89.99, amount_minor: 8999, currency: 'GBP' }); // "£89.99"
formatPrice({ amount_major: 89.99, amount_minor: 8999, currency: 'EUR' }); // "€89.99"
```

`Intl.NumberFormat` handles the currency symbol, decimal separator, and grouping automatically based on the locale you pass in. Passing `undefined` uses the browser's current locale.

---

## Specifying a locale

If your integration targets a specific market or you want consistent formatting regardless of browser locale, pass a locale explicitly:

```js
function formatPrice(money, locale = 'en-GB') {
  return new Intl.NumberFormat(locale, {
    style: 'currency',
    currency: money.currency,
  }).format(money.amount_major);
}

formatPrice({ amount_major: 89.99, amount_minor: 8999, currency: 'GBP' }, 'en-GB'); // "£89.99"
formatPrice({ amount_major: 89.99, amount_minor: 8999, currency: 'EUR' }, 'de-DE'); // "89,99 €"
formatPrice({ amount_major: 89.99, amount_minor: 8999, currency: 'EUR' }, 'fr-FR'); // "89,99 €"
formatPrice({ amount_major: 89.99, amount_minor: 8999, currency: 'EUR' }, 'nl-NL'); // "€ 89,99"
```

The locale you pass to `Intl.NumberFormat` can match the customer's market or the `accept-language` value you use when calling the API. For example, a customer may see German product content and a EUR price formatted for Germany, while another market may use different price formatting for the same currency.

---

## Server-side formatting

If you're rendering prices server-side (Node.js), `Intl.NumberFormat` works the same way - it's part of the JavaScript runtime, not browser-specific:

```js
const formatted = new Intl.NumberFormat('en-GB', {
  style: 'currency',
  currency: 'GBP',
}).format(89.99);
// "£89.99"
```

---

## Displaying discounted prices

When a product has a discount, `pricing.non_discounted` and `pricing.total` are both returned. Show both to communicate the saving:

```js
function formatPrices(pricing, locale) {
  const format = (money) => new Intl.NumberFormat(locale, {
    style: 'currency',
    currency: money.currency,
  }).format(money.amount_major);

  if (pricing.total.amount_minor < pricing.non_discounted.amount_minor) {
    return {
      display: format(pricing.total),
      was: format(pricing.non_discounted),
    };
  }

  return { display: format(pricing.total) };
}
```

When `pricing.total.amount_minor === pricing.non_discounted.amount_minor`, no discount applies - show only the price, not a crossed-out was price.

---

## Price Lock

Both parking search endpoints return `price_lock_valid_until` for every parking result. Use it to understand the Price Lock window for the `pricing.total` shown in search.

`price_lock_valid_until` and `product_token_valid_until` are separate values. See [Price Lock](../integration-guides/07-price-lock.md) for how they work together and the options available in an exceptional changed-price response.

---

## Related

- [Price Lock](../integration-guides/07-price-lock.md) - how the selected price is protected through checkout
- [Selling Parking](./selling-parking.md) - where and how to display prices in the product listing
- [GET /v2/products/parking/detailed](../api-reference/get-parking-availability-detailed.md) - the `pricing` object in the search response
