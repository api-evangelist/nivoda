---
name: Create a Nivoda diamond request by certificate number
description: Raise a diamond request (media / info / concierge) by certificate number, optionally auto-creating an order.
api: https://integrations.nivoda.net/api/diamonds
operations:
  - authenticate.username_and_password
  - create_request
---

# Create a diamond request

Use `create_request` when you have a certificate number and want Nivoda to confirm details
(e.g. media, no BGM, eye-clean) and optionally turn it into an order. Requires Pro access.
Authenticate first and send `Authorization: Bearer <token>`.

```graphql
mutation {
  create_request(
    certificate_number: "42717505123",
    lab: GIA,
    comment: "Please confirm if NO BGM and eyeclean",
    customer_order_number: "Ref 24-ZX",
    create_order: true,
    requested_info: [MEDIA]
  )
}
```

`create_request` returns the id of the created request.

## Rules
- `certificate_number` is **required**; `lab` identifies the grading lab (e.g. `GIA`).
- `customer_order_number` shows up on the invoice if the stone is purchased and delivered.
- `create_order: true` auto-creates an order from the request when the stone is confirmed.
- `requested_info` selects what to ask for (e.g. `[MEDIA]`).
- Check the GraphQL `errors[]` array before treating the request as created.
