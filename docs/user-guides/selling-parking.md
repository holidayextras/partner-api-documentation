## What is airport parking?

Airport parking lets travellers leave their car near the airport while they fly. There are several different service models - from leaving your car with a driver who parks it for you, to driving yourself to a nearby car park and catching a shuttle. We surface all of these as products through the API.

Holiday Extras knows the parking products, the content, and what tends to convert. You know your customers and where parking fits in their booking journey. Work with our team when shaping your parking display so we can help you use the right content, ordering, and messages for the strongest attach rate.

---

## Airport parking customer messages

The strongest airport parking messages give customers a clear reason to book before they travel. Use them in your UI headers, product cards, CTAs, and emails.

| Message | Where to use |
|---|---|
| Pre-book & save up to 75% | Search entry point or intro banner where the saving claim applies |
| Leave your car in safe hands | Product listing, trust message, reassurance copy |
| Park Mark secure | Trust badge alongside products with Park Mark accreditation |

---

## Display best practices

How you present parking has a direct impact on how many customers add it. The right approach depends on where parking sits in your customer journey.

---

### Holiday or flight booking partners

For partners selling parking as part of a wider holiday, flight, ferry, or travel booking journey, parking is usually an in-path extra. The customer hasn't specifically gone looking for parking - you're introducing it at the right moment.

**Show parking as a default step, not behind a click.** Displaying it in-path significantly outperforms presenting it as an opt-in extra hidden behind a button.

**Show 3 products, curated by Holiday Extras.** Holiday Extras controls and optimises product ordering on your behalf, surfacing the three options most likely to convert for that airport and customer. Presenting these as the default selection - with a "view more" option for customers who want to explore - gives the best attachment rates.

**Keep cards scannable.** Customers are mid-booking and moving quickly. A clear service type label (`parking_type`), the selling point, key transfer details for park & ride, and a prominent price is enough to drive a decision.

**Use content to answer the quick question: "Is this the right parking option?"** Keep the product card focused on:

- `name` or `site_name` - the car park name the customer will recognise later
- `parking_type` - a plain service label such as "Park & Ride" or "Meet & Greet"
- `selling_point` - the short reason to choose the product
- `transfer_duration_minutes` and `transfer_frequency_minutes` for park & ride
- `distance_miles` for park & stroll and close-to-terminal products
- `desktop_image` or `mobile_image` where your layout has space for one clear image
- covered parking (`is_under_cover`), EV charging, or vehicle-size information when it is likely to affect the customer's choice
- cancellation policy from the search response, shown next to the price

Save longer content fields such as `information`, `directions`, `arrival_procedures`, and `departure_procedures` for the detail view or post-booking experience. In this journey, too much copy can slow the customer down.

For the complete list of content fields, including what each field contains and where it is typically used, see the [parking content field guide](../api-reference/get-parking-content.md#response-fields).

**Suggested card layout (park & ride):**
```
[Product name]          [parking_type label]
[selling_point]
Transfer every [transfer_frequency_minutes] mins · [transfer_duration_minutes] min to terminal
[cancellation_terms label]
                                    [total_amount]  SELECT
```

**Suggested card layout (meet & greet):**
```
[Product name]          MEET & GREET
[selling_point]
No shuttle required · Drive straight to the terminal
[cancellation_terms label]
                                    [total_amount]  SELECT
```

---

### Parking comparison and aggregator partners

For partners selling parking as a standalone travel ancillary or comparison experience, the customer has arrived specifically to find and compare airport parking. They're in research mode and want to see the full picture before committing.

**Show all available products.** Breadth of inventory is a core part of the value - customers expect to see everything on offer and compare their options directly.

**Invest in the detail view.** Because customers are actively comparing, richer product information converts better here. A full-page or expanded product view with address, distance, directions, facilities, images, and arrival procedures gives customers the confidence to book.

**Surface filters and sorting.** With a full product list, letting customers filter by service type (`parking_type`), price, or distance - and sort by any of these - makes the experience significantly more useful.

**Use content to support comparison.** In this journey, customers are weighing up trade-offs, so give them more detail:

> **Review data availability:** Parking review fields are included in content responses. The rating value, review count, and summary are nullable, so they return `null` when review data is not available for a car park.

- Use `site_name` to group variants of the same car park, then show the meaningful difference between options, such as cancellation flexibility, EV charging, or vehicle size.
- Use `image_gallery` on product detail pages, with `desktop_image` or `mobile_image` as the listing image.
- Show address, terminal, distance, transfer duration, transfer frequency, directions, and facilities so customers can compare convenience as well as price.
- Surface trust and suitability signals such as `is_vehicle_parked_for_you`, `is_under_cover`, `maximum_vehicle_size`, and the EV charging fields. Security and accessibility detail, where a car park provides it, comes through in `info_tab`.
- Use `product_requirements` from the search response to explain what the customer will need to provide, and whether it is needed at booking or before entry.
- Include `arrival_procedures` and `departure_procedures` in the detail view when they help explain how the service works, especially for meet & greet and return greet.
- Show the rating, review count, and review summary when present. Treat review values as optional because they may be unavailable for some sites.
- Show cancellation policy clearly before the customer selects the product, including non-refundable options.

**Suggested card layout (aggregator):**
```
[Product image]
[Product name]                      [parking_type label]
[selling_point]
[distance_miles] from terminal · Transfer every [transfer_frequency_minutes] mins
[reviews.rating] ([reviews.count] reviews)   [cancellation_terms label]
                                    [total_amount]  VIEW DETAILS
```

---

### Airlines

For airlines, parking works best when it feels connected to the flight. The customer is already thinking about the airport, terminal, and travel dates - use that context to make parking feel like a natural part of the trip.

**Use flight details to simplify the search.** Send the outbound departure time, inbound arrival time, and outbound flight number when available. The API uses the flight times to calculate the parking window and the flight number to filter products for the relevant terminal. This means customers do not need to identify their terminal or work out suitable parking times themselves.

When using flight times, send the return flight's scheduled arrival time at the airport where the customer parked their car. For example, for a Gatwick to Alicante return trip, the inbound arrival time is when the customer lands back at Gatwick. The API uses that time to derive the parking exit window.

**Keep the choice simple in-path.** If parking appears during flight booking, check-in, or manage booking, show a small set of recommended products with clear differences: service type, transfer time, cancellation flexibility, and total price.

**Keep parking visible in the wider trip flow.** Airlines often have strong check-in and manage-booking journeys. Link customers back to parking there, so the product stays close to the flight rather than feeling like a separate add-on.

---

## Grouping products with `site_name`

Some products are variations of the same car park - same location, same service, but with differences a customer would want to choose between. Products sharing the same `site_name` are variants of the same site.

`site_name` is returned in the product content and serves two purposes: it's the grouping key for your UI logic, and the display name to show the customer. There's no need to map to a separate label - whatever is in `site_name` is what the customer sees.

**Use this to build a better UI.** Instead of showing multiple cards that look nearly identical, group them into a single card and let the customer pick the option that suits them - like a cabin class selector. This reduces visual noise, makes comparison easier, and puts the meaningful choice front and centre.

Common grouping scenarios:

**Cancellation flexibility** - the same car park offered at a flexible rate with free cancellation, and a non-flexible rate without:

```
┌────────────────────────────────────────────────┐
│ APH Park & Ride Gatwick                        │
│ Park & Ride · 10 min shuttle every 15 min      │
│                                                │
│  ● Flexible        £89.99   Free cancellation  │
│  ○ Non-flexible    £74.99   Non-refundable     │
│                                                │
│                                 [SELECT]       │
└────────────────────────────────────────────────┘
```

**EV charging** - the same car park with and without electric vehicle charging included:

```
┌─────────────────────────────────────────────┐
│ Long Stay South Gatwick                     │
│ Park & Ride · 8 min shuttle every 10 min    │
│                                             │
│  ● Standard        £74.99                  │
│  ○ With EV charging £89.99  ⚡ Charging included│
│                                             │
│                              [SELECT]       │
└─────────────────────────────────────────────┘
```

**Vehicle size** - the same car park with a variant for larger vehicles such as SUVs or vans:

```
┌─────────────────────────────────────────────┐
│ BCP Airport Parking Birmingham              │
│ Park & Ride · 5 min shuttle every 20 min    │
│                                             │
│  ● Standard        £64.99                  │
│  ○ Larger vehicle  £79.99   SUVs & vans     │
│                                             │
│                              [SELECT]       │
└─────────────────────────────────────────────┘
```

Products without a shared `site_name` should be shown as separate cards as normal.

---

## The five product types

The `parking_type` field in the content API tells you which type of product you are selling. The first four describe how the service works, and each has a different customer journey - displaying this clearly helps customers know what to expect and choose with confidence. `economy_parking` works differently: it describes the product's positioning rather than its mechanics.

### Meet & Greet (`meet_and_greet`)

The customer drives to the airport's drop-off zone. A uniformed chauffeur meets them, checks the car in, and drives it to a secure compound. On return, the chauffeur brings the car back to the same spot.

**Best for:** Customers who want the most convenient experience. Popular for families with lots of luggage.

**What to tell customers:**
- Drive straight to the terminal drop-off - no car park to find
- A driver will be waiting with a sign
- Allow a few extra minutes at the terminal pick-up lane

**Key content fields to surface:**
- `arrival_procedures` - exact instructions for the drop-off
- `departure_procedures` - where to meet the driver on return

---

### Return Greet (`return_greet`)

Similar to meet & greet, but the chauffeur only meets the customer on return, not on departure. The customer parks themselves when they arrive.

**Best for:** Customers who don't mind parking on arrival but want to be met on return.

---

### Park & Stroll (`park_and_stroll`)

The customer parks in a facility very close to the terminal - usually an official airport car park or a nearby lot within walking distance. No shuttle needed.

**Best for:** Customers who want reliability and simplicity. Short walks, no waiting for a bus.

**Key content fields to surface:**
- `distance_miles` - walking distance to the terminal
- `directions` - how to find the car park

---

### Park & Ride (`park_and_ride`)

The customer drives to an off-airport car park and catches a shuttle bus to the terminal. The most common and typically most affordable option.

**Best for:** Price-conscious travellers happy to allow extra time for the shuttle.

**Key content fields to surface:**
- `transfer_duration_minutes` - how long the shuttle takes
- `transfer_frequency_minutes` - how often the shuttle runs
- `operating_time_earliest` / `operating_time_latest` - first and last shuttle times
- `transfer_overview` - a prose summary of the transfer provision and its constraints, including any period of the stay when the shuttle doesn't run

> **Transfer frequency and time are the most important piece of information for customers choosing between park & ride options** - keeping these visible on the card makes a real difference to conversion.

---

### Economy parking (`economy_parking`)

A budget-friendly option for price-conscious travellers, focused on affordability rather than premium quality.

Unlike the other values, this one does not tell you how the service works. An economy product may be park & stroll, park & ride, valet, or reached by public transport, so read the content fields to find out how a particular product operates rather than assuming a shuttle.

**Best for:** Customers led primarily by price who are happy to trade convenience for it.

**Key content fields to surface:** `transfer_overview` for how the customer reaches the terminal, and `distance_miles`, `transfer_duration_minutes`, `transfer_frequency_minutes` and `operating_time_earliest` / `operating_time_latest` wherever they are populated. Because the service varies between economy products, surface these fields when present rather than reserving space for them on every card.

---

## What to display on your product listing

These are the most important fields when customers are comparing options:

| What to show | API field | Example |
|---|---|---|
| Product name | `name` | "APH Park & Ride Gatwick" |
| Service type | `parking_type` | "Park & Ride" |
| Price | `pricing.total.amount_major` (from search) | €74.99 |
| Terminal | `terminal` | "South Terminal" |
| Distance | `distance_miles` | "0.3 miles from terminal" |
| Shuttle time | `transfer_duration_minutes` | "10 min shuttle" |
| Shuttle frequency | `transfer_frequency_minutes` | "Every 15 min" |
| Cancellation policy | `policies` (from search) | "Free cancellation" |
| Selling point | `selling_point` | "Best value at Gatwick" |
| Reviews | `content.reviews.rating.value`, `content.reviews.count`, `content.reviews.summary` | "Rated 8.9/10 from 1,250 reviews - Customers love the smooth drop-off and quick transfer" |
| Photo | `desktop_image` / `mobile_image` | Product image |
| EV charging | `is_electric_charging_available` | "EV charging available" |

---

## Caching product content

You can cache product content returned by `GET /v2/content/parking`, keyed by product code and language. Names, descriptions, images, directions, facilities, and procedures change less often than prices and availability, so caching them can make high-traffic integrations faster.

Keep search results fresh. Prices, availability, cancellation policy, and `product_requirements` depend on the customer's dates, selected product, or booking state. Fetch these from the search or booking response rather than from a content cache.

---

## Plan when to collect essential details

Some car parks need extra information to prepare for the customer's arrival. This might include a mobile number, flight number, vehicle registration, or details such as the vehicle make, model, and colour.

Each detail supports a practical part of the parking journey. A licence plate can be used for barrier access. For meet and greet or valet parking, the vehicle make, model, and colour help staff identify the car, while a mobile number lets the driver coordinate the handover with the customer. Flight details help operators adjust arrangements when a flight is delayed. They also help park and ride operators anticipate demand and plan shuttle buses during disruption.

The API tells you which details are needed and when, including any deadline for information that can be added after booking:

- **Required at booking:** collect the information before the customer completes their booking.
- **Required before travel:** give the customer an easy way to add the information after booking and use the returned deadline to decide when to remind them.

### Vehicle registration

An increasing number of car parks are using licence plate recognition to identify vehicles on arrival and provide access. Depending on the product, the vehicle registration may be required at booking or before travel.

You can collect it in your own checkout, app, account area, or wherever customers manage their booking. Customers can also add it through the Holiday Extras hosted confirmation page, or your customer service team can enter it through the Customer Service Portal.

See [Accessing parking](./accessing-parking.md#collecting-an-essential-vehicle-registration-after-booking) for the collection routes and technical guidance after booking.

---

## Displaying product photos

The content API returns three image formats per product:

| Field | Use case |
|---|---|
| `desktop_image` | Full-size product listing image |
| `mobile_image` | Mobile-optimised crop |
| `wallet_image` | Small thumbnail (booking confirmation, wallet passes) |
| `image_gallery` | Complete gallery of product images for a detail page |

`desktop_image`, `mobile_image`, and `wallet_image` are URI strings; `image_gallery` is an array of URI strings. Car park imagery is typically landscape - respecting the aspect ratios keeps the listing looking its best.

---

## Vehicle handling

These content fields help customers understand how the car park looks after their vehicle:

| Field | What it tells the customer |
|---|---|
| `is_vehicle_parked_for_you` | Whether staff park the vehicle, or the customer parks it themselves |
| `is_under_cover` | Whether the parking is under cover |
| `info_tab` | Longer-form site information, including security features and accessible facilities where the car park provides them |

Surface these details where available, particularly when they may affect the customer's choice.

---

## Pricing tips

- **Show the total price** - once a customer has entered their dates, showing the exact price for their stay makes it easy to commit.
- **Use our product ordering** - Keeping our recommended ordering as the default gives customers the best selection.
- **Flag non-refundable products clearly** - showing cancellation flexibility prominently alongside the price helps customers make an informed choice and reduces cancellation contacts down the line.
