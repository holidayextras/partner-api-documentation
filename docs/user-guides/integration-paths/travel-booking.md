# Selling parking as part of a wider travel booking

Use this path when parking is offered alongside another travel product, such as a holiday, flight, hotel, ferry, package, or wider travel checkout.

In this journey, the customer is usually focused on the main travel purchase. Parking works best when it feels like a helpful next step rather than a separate research task.

---

## What to focus on

| Focus | Why it helps | Useful docs |
|---|---|---|
| Keep parking in the main booking path | Customers are more likely to add parking when it appears at a natural point in the booking flow | [Customer journey](../customer-journey.md) |
| Use the travel details you already hold | Flights, airport, dates, times, language, and currency help pre-fill the search and return the most relevant results | [API overview](../../integration-guides/01-api-overview.md) |
| Keep search fast | Cached product content plus live price and availability calls help protect the performance of the wider booking flow | [Search endpoints](../../integration-guides/04-search-endpoints.md) |
| Keep the choice compact | Customers are mid-checkout and need quick confidence | [Selling parking](../selling-parking.md) |
| Keep the main purchase moving | Price Lock and token-based booking help customers add parking without holding up the wider holiday or travel purchase | [Search endpoints](../../integration-guides/04-search-endpoints.md) |
| Keep access details easy to find later | Customers need the latest arrival instructions close to travel | [Accessing parking](../accessing-parking.md) |

---

## Suggested shape

Show parking after the customer has chosen the main travel product and before final payment. Use the travel details you already hold, including flight details where available, to pre-fill the search and get the most relevant results.

For most in-path journeys, a compact list of recommended products is enough. Holiday Extras will set the recommended products for your integration and brand, so the default list is tuned for the customers and route you are serving.

Show service type, selling point, transfer time or walking distance, and total price. Give customers a route to more detail, but keep the default decision quick to scan. If parking is part of a package, the customer's cancellation terms may be covered by the wider package conditions, so only surface parking-specific cancellation detail where it is relevant to your journey.

After booking, include the Holiday Extras hosted confirmation page link in your own confirmation email or account area. If you render parking details inside your own experience, refresh the booking before sending travel reminders.

---

## Integration notes

1. Cache `GET /v2/locations` daily to power airport selection and supported currencies.
2. Cache product content with `GET /v2/content/parking` where you need the fastest possible in-path search.
3. Fetch live price and availability with `GET /v2/products/parking` for each customer search.
4. Display the recommended products agreed for your integration and brand.
5. Collect the fields required at booking. Vehicle registration can often be collected after purchase where the API marks it as needed before travel.
6. Book with `POST /v2/bookings/parking` using the selected `product_token`.
7. Fetch `GET /v2/bookings/parking/{ref}` and store the booking reference and confirmation page link.
8. Use amendment and cancellation quote flows if customers or service teams need to manage changes.
