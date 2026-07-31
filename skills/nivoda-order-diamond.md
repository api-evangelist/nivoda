---
name: Order or hold a diamond on Nivoda
description: Find a diamond, place a temporary hold, then create an order using its offer id (Nivoda API Pro).
api: https://integrations.nivoda.net/api/diamonds
operations:
  - authenticate.username_and_password
  - diamonds_by_query
  - create_hold
  - create_order
---

# Hold and order a diamond (Pro)

Ordering, holds, requests and concierge requests require **Nivoda API Pro** access.
Authenticate first (see `nivoda-search-diamonds.md`) and send `Authorization: Bearer <token>`.

## 1. Find the diamond and its offer id

Run `diamonds_by_query` and read `items[].id` / `items[].diamond.id`. An **offer id** has
the form `DIAMOND/<diamond_id>`, e.g. `DIAMOND/8bc1e15f-7086-489e-a79f-7e76a6063329`.

## 2. (Optional) Place a hold

```graphql
mutation {
  create_hold(ProductId: "8bc1e15f-7086-489e-a79f-7e76a6063329", ProductType: Diamond) {
    id
    denied
    until
  }
}
```

`ProductId` is the bare diamond id (no `DIAMOND/` prefix); `ProductType` is `Diamond`.
Keep the returned `id` to confirm or release the hold later. `denied: true` means the
hold was refused; `until` is the expiry.

## 3. Create the order

```graphql
mutation {
  create_order(
    items: [
      {
        offerId: "DIAMOND/8bc1e15f-7086-489e-a79f-7e76a6063329",
        customer_order_number: "Ref 24-ZX",
        customer_comment: "Please confirm no black in table",
        return_option: true
      }
    ]
  )
}
```

`create_order` returns the created order id.

## Rules
- Only `offerId` is required per item; everything else is optional.
- `customer_order_number` is your own reference and appears on the dashboard and invoice —
  use it to de-duplicate, since the API has **no idempotency-key** mechanism.
- `return_option: true` is honored only for returnable stones; on a non-returnable stone it
  silently becomes `false`.
- Confirm success via the returned order id and check the GraphQL `errors[]` array.
