# Accessing parking: keeping customers informed after booking

Once a customer has booked, they need a clear place to find their booking details and any information they will need before travel. Some details are available immediately. Others may be refined as the supplier completes the booking.

This guide starts with the post-booking journey that applies across products, then covers the access information specific to Airport Parking.

---

## The customer timeline

The information customers need, and how you share it, changes through the booking journey.

| Stage | What the customer needs | What to plan for |
|---|---|---|
| Immediately after booking | Confirmation that the booking succeeded, a booking reference, the practical details already available, and a reliable place to find the latest information | Send or display the confirmation using the communication approach agreed for your integration |
| After an amendment or cancellation | A clear record of what changed, including any price or refund outcome | Make sure the customer receives an updated confirmation or cancellation message |
| As supplier information becomes available | Any updated instructions, supplier references, or access information | Holiday Extras-managed pages and emails update automatically. If you manage the information yourself, fetch the latest booking state |
| A few days before travel | The latest practical information, including where to go, when to arrive, how to use the product, and who to contact | Surface these details again when they are most relevant, for example alongside check-in prompts or other holiday communications |

---

## Choose how customers receive updates

Choose the approach that best fits your customer journey. You can use our hosted confirmation page, enable Holiday Extras-generated emails, or bring the latest details into your own channels. Some partners include the hosted page in a simple booking confirmation, while others keep the complete experience within their own app, account area, or emails.

Whichever approach you choose, give the customer one clear and reliable route to their latest booking information.

### Holiday Extras hosted confirmation page

The hosted confirmation page is our recommended option for the most hassle-free integration. Include its link in your own booking confirmation, app, account area, or wherever customers manage their booking. Holiday Extras will keep the page current for you.

When you view a parking booking through the API, the response includes a link to the page we host for your customer. See [View a parking booking](../api-reference/get-parking-booking.md) for the endpoint and response field.

For parking, the page includes the latest booking information held by Holiday Extras, such as access instructions, the car park address, directions, arrival and departure procedures, and contact details. If the operator refines instructions, or a barcode or supplier reference arrives later, the page updates without you needing to replace the link or send another message.

### Holiday Extras-generated confirmation emails

You can choose whether Holiday Extras sends booking emails on your behalf. If you enable them, we automatically email the customer when a booking is created, amended, or cancelled. Each email contains the information the customer needs for that stage of the journey.

Before enabling these emails, consider how they fit alongside your own booking and travel communications. You may still want to mention parking within a wider confirmation, with each message having a clear purpose in the customer journey.

### Partner-owned channels

You can present confirmation details through your own email templates, app, account area, customer dashboard, or wherever customers manage their booking. This gives you full control of when and how information appears.

If your channel or internal systems need to respond when supplier information changes, a `booking.impacted` webhook can prompt you to fetch the latest booking state. This supports event-driven updates without requiring regular polling. See the [webhook integration guide](../integration-guides/05-webhooks.md) for implementation guidance.

---

## If you manage the customer-facing information yourself

Refresh the booking at the points where current information matters, such as after an amendment or before a reminder. A planned fetch at those moments may be enough for your journey. Webhooks can provide more immediate timing when your systems need it.

For Airport Parking, these fields are the most likely to change after booking:

| Field | Why it might update |
|---|---|
| `access_methods` | The available access methods or their order may update after the supplier confirms the booking. A barcode value is returned immediately as a stable image URL. A reference may arrive later, while `supplier_confirmation` has no value because the supplier contacts the customer directly |
| `supplier_fulfilment.status` | Supplier fulfilment moves from `pending` to `fulfilled` once supplier processing completes, or to `failed` if fulfilment cannot be completed |
| `supplier_reference` | The supplier may add its own booking reference after confirming the booking |
| `product_content.arrival_procedures` / `product_content.departure_procedures` | The operator may refine the instructions before travel |

The view-booking response also includes `supplier_fulfilment.expected_completion_datetime`. This is an estimate of when supplier processing is expected to complete and is most useful for internal timing rather than customer-facing information.

If `supplier_fulfilment.status` is `failed`, the supplier could not complete the original fulfilment. The booking remains active while Holiday Extras handles the rebooking on your behalf.

If you manage the customer-facing information yourself, refetch the booking after a subsequent update to pick up the latest product, access, and supplier details. A `booking.impacted` webhook can prompt this refresh where webhooks are part of your integration. Follow your agreed support process if the customer needs an update while rebooking is in progress.

---

## Parking access methods

If you use the hosted confirmation page or Holiday Extras-generated emails, Holiday Extras presents the relevant access information for you. If you display parking details in your own channels, use `access_methods` from the view-booking response as the source for how the customer accesses the car park. See [View a parking booking](../api-reference/get-parking-booking.md) for the endpoint details.

The API returns only the methods that apply to the booked product. They are ordered by priority, with `priority: 1` shown first. Remaining methods can be shown as backup guidance where that is useful.

Each method contains:

```json
{
  "priority": 1,
  "type": "barcode",
  "value": "https://images.holidayextras.com/barcode/2f4d7f8c-4d5e-4e9a-a2ce-38f6c1c84b4a"
}
```

The meaning of `value` depends on the method type:

| `access_methods.type` | What it means | What to show the customer |
|---|---|---|
| `licence_plate` | The car park uses the customer's vehicle registration for access | Show the vehicle registration the barrier or operator will use. If it has not been provided, collect it before the relevant `product_requirements[].requirement_deadline` |
| `barcode` | The car park uses a barcode for access | Render the URL in `value` as an image |
| `reference` | The car park uses a text reference for access | Display the returned reference prominently. It may be `null` until supplier fulfilment completes |
| `supplier_confirmation` | The supplier sends access instructions directly, often by email | Tell the customer to follow the supplier's instructions. The value is `null` because the instructions arrive separately |

### Barcode image URLs

For a barcode method, `access_methods[].value` is a stable image URL. This makes it easy to add the barcode to an email, app, account area, or dashboard as soon as the booking is created. Use the URL as the source for an image, rather than showing the URL itself or generating a separate barcode.

Before supplier fulfilment completes, an image request to the URL may return `404 Not Found` with `Cache-Control: no-store`. The value in the booking response is still valid. Once the supplier confirms the booking, the same URL returns the PNG image. One URL therefore works throughout the journey, with nothing to replace in an email, dashboard, or stored booking record.

If you proxy or pre-fetch images, follow the response caching headers. The `no-store` response allows the image to be requested again once it is ready. Where useful, `confirmation_page_link` gives the customer another route to their latest details if an email client blocks images or an image cannot be displayed.

In an email or web view:

```html
<img
  src="{{ access_method.value }}"
  width="300"
  height="300"
  alt="Parking access barcode"
>
```

### Supplier confirmation and references

`supplier_confirmation` is different from a reference that has not arrived yet. It means the supplier sends the access instructions outside the Partner API, usually by email. Keep the customer's email address accurate and explain in your own confirmation that the supplier will contact them directly.

For a `reference` method, wait until `value` is present before showing it as an access reference. Once `supplier_fulfilment.status` is `fulfilled`, the value is the supplier reference or, where the supplier has not provided one, the Holiday Extras booking reference returned for that method.

---

## What to show in your own channels for parking

This section covers the parking information to include when you render confirmation details in your own emails, app, account area, or wherever customers manage their booking. View the booking through the API at the point you generate or display information that needs to be current. See [View a parking booking](../api-reference/get-parking-booking.md) for the endpoint details.

### Immediately after booking

If you use the hosted confirmation page, your own confirmation can stay simple: confirm the booking, show the booking reference, and give the customer the link to their latest details.

If you render the details yourself, show or send:

- Holiday Extras booking reference: `booking_reference`
- Arrival and exit date and time: `parking_entry_datetime` and `parking_exit_datetime`
- Car park name and address: `product_content.name` and `product_content.address`
- Directions: `product_content.directions`
- Contact number: `product_content.telephone`
- Access methods: show the highest-priority method first where its customer-facing value is available

For a barcode method, use the image URL immediately. For `supplier_confirmation`, tell the customer to follow the supplier's direct instructions. A reference may not be available until supplier fulfilment completes. You can keep `confirmation_page_link` as an additional route to the hosted details where it adds value to your journey.

### After an amendment or cancellation

After a successful amendment, refetch the booking before showing or sending the updated confirmation. Access methods, access values, procedures, and requirement deadlines may have changed. Make the new details clear so the customer knows which information now applies.

For a cancellation, confirm the cancellation and any confirmed refund outcome in the channel the customer uses to manage the booking.

If Holiday Extras-generated emails are enabled, your customer receives these messages automatically. Plan any messages you send alongside them so that each one has a clear purpose in the customer journey.

### A few days before travel

If you send a reminder or surface the details again before travel, fetch the booking again so the customer sees the latest information. For parking, around three days before `parking_entry_datetime` is a useful point to remind the customer, although the timing should fit the wider journey.

Include:

- Booking and supplier references, where available
- Access methods, showing the highest-priority method first
- Arrival and exit date and time
- Car park name, address, directions, and contact number
- Arrival and departure procedures
- Additional product information that affects the journey

Where it complements your own channel, you can also include `confirmation_page_link` as a reliable route to the latest hosted details.

If the car park needs more information before arrival, such as a vehicle registration, check the relevant `product_requirements[].requirement_deadline` before contacting the customer. Use that deadline rather than a fixed rule for every booking.

### Collecting an essential vehicle registration after booking

Where vehicle registration is marked as `required_at: "before_travel"`, it must be added by `product_requirements[].requirement_deadline`. You can choose the collection route that best fits your customer journey:

- Collect it through your own email, app, account area, or wherever customers manage their booking, then submit it using the amendment endpoint.
- Use the Holiday Extras hosted confirmation page, where the customer can add it themselves.
- Ask your customer service team to add it manually through the Customer Service Portal.

After the registration is added, refetch the booking to confirm the saved value and pick up any updated `licence_plate` access method. See [View a parking booking](../api-reference/get-parking-booking.md) for the current requirement and vehicle details. If you collect the registration through your own channels, use the [parking amendment flow](../api-reference/patch-parking-amendment-quote.md) to submit it.

---

## Parking journeys on the day of travel

The exact journey depends on the parking service.

### Park and ride

**Outbound:**

1. The customer drives to the car park address.
2. They follow the highest-priority access method or the supplier's direct instructions.
3. They park the car and either keep their keys or hand them to the operator, following the arrival procedures.
4. They take the shuttle to the terminal.

**Return:**

1. The customer follows the departure procedures and takes the shuttle from the terminal.
2. They collect their keys where required and leave the car park.

### Park and stroll

**Outbound:**

1. The customer drives to the car park and follows the highest-priority access method or the supplier's direct instructions.
2. They park the car and follow the arrival procedures for their keys.
3. They walk to the terminal.

**Return:**

1. The customer walks back to the car park, following the departure procedures.
2. They collect their keys where required and leave the car park.

### Meet and greet

**Outbound:**

1. The customer follows the arrival procedures and drives to the agreed terminal meeting point.
2. They hand the keys to the driver.

**Return:**

1. The customer follows the departure procedures and contacts the operator when instructed.
2. The driver returns the car at the agreed meeting point.

### Return greet

**Outbound:**

1. The customer drives to the car park, follows the access instructions, and parks the car.
2. They continue to the terminal as described in the arrival procedures.

**Return:**

1. The customer follows the departure procedures and contacts the operator when instructed.
2. The driver returns the car at the agreed meeting point.

---

## Quick reference

| What the customer needs | Field | When it is available |
|---|---|---|
| Holiday Extras booking reference | `booking_reference` | Immediately |
| Hosted confirmation page | `confirmation_page_link` | Immediately |
| Arrival and exit date and time | `parking_entry_datetime`, `parking_exit_datetime` | Immediately |
| Car park name, address, and contact number | `product_content.name`, `product_content.address`, `product_content.telephone` | Immediately |
| Directions | `product_content.directions` | Immediately; may be refined later |
| Arrival and departure procedures | `product_content.arrival_procedures`, `product_content.departure_procedures` | Immediately; may be refined later |
| Parking access methods | `access_methods` | Returned in priority order. The barcode URL is returned immediately; other values may update later |
| Supplier reference | `supplier_reference` | May arrive after supplier processing |
| Deadline for information needed before travel | `product_requirements[].requirement_deadline` | Refetch before reminders and after an amendment that changes the parking entry date |

All fields come from the view-booking response unless stated otherwise. See [View a parking booking](../api-reference/get-parking-booking.md) for the endpoint details.

---

## Related guides

- [Selling Parking](./selling-parking.md) - guidance on presenting and selling parking
- [Parking content field guide](../api-reference/get-parking-content.md#response-fields) - the complete list of content fields and where they are typically used
- [View a parking booking](../api-reference/get-parking-booking.md) - the full booking response and access-method behaviour
- [Webhooks](../integration-guides/05-webhooks.md) - payload, signing, retry, and refetch guidance
- [FAQs](../../faqs.md) - payment, refunds, and customer-support responsibilities
