---
name: Authenticate and search Nivoda diamond inventory
description: Log in to the Nivoda GraphQL API and search live natural / lab-grown diamond inventory with filters, pagination and ordering.
api: https://integrations.nivoda.net/api/diamonds
operations:
  - authenticate.username_and_password
  - diamonds_by_query
---

# Search Nivoda diamond inventory

Nivoda exposes a single GraphQL endpoint. All requests are `POST` with
`Content-Type: application/json` to `https://integrations.nivoda.net/api/diamonds`
(staging: `https://intg-customer-staging.nivodaapi.net/api/diamonds`).

## 1. Authenticate

Run the login query using your Nivoda platform username and password. It returns a
session token.

```graphql
{
  authenticate {
    username_and_password(username: "you@example.com", password: "••••") {
      token
    }
  }
}
```

Read the token from `data.authenticate.username_and_password.token`.

## 2. Send the token

On every subsequent request add the header `Authorization: Bearer <token>`.
(In the GraphiQL explorer instead wrap the operation in `as(token: $token) { ... }`.)

## 3. Search diamonds

Call `diamonds_by_query`. `limit` is **50 maximum**; page with `offset`.

```graphql
query {
  diamonds_by_query(
    query: { labgrown: false, shapes: ["ROUND"], sizes: [{ from: 1, to: 1.5 }],
             has_v360: true, has_image: true, color: [D, E] },
    offset: 0, limit: 50,
    order: { type: price, direction: ASC }
  ) {
    items {
      id
      price
      discount
      diamond {
        id image video availability supplierStockId
        certificate { certNumber lab shape carats color clarity cut polish symmetry }
      }
    }
    total_count
  }
}
```

## Rules
- Request only the fields you need — the API returns exactly the selected fields.
- `total_count` drives pagination; increment `offset` by `limit` until you have `total_count` rows.
- Image/video URLs are resizable by changing the `500/500` dimensions in the URL.
- Accepted `shapes` values are listed in `examples/accepted_shapes.txt` (~50 values).
- Errors arrive in the GraphQL top-level `errors[]` array; check it before reading `data`.
