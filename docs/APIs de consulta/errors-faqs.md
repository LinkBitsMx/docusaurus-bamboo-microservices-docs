---
sidebar_position: 5
title: Errors and FAQs
---

# Errors and FAQs

## Response codes

| Code | Meaning | Recommended response |
| --- | --- | --- |
| `200` | Successful query | Process the JSON response |
| `401` | Missing or invalid API Key | Check credentials |
| `404` | Resource not found | Validate the folio, SKU, or ticket |
| `500` | Internal error | Report with request details |

## Best practices

- Do not share the API Key in public channels.
- Do not store the API Key in repositories.
- Do not send the API Key in the URL.
- Use HTTPS in production environments.
- Log the queried folio, endpoint, date, time, and HTTP code internally.

## FAQs

### Where do I get the API Key?

It is provided by the team responsible for the integration.

### Can the API Key be used from frontend code?

It is not recommended. It should be used from a backend, private service, or server controlled by the integrator.

### Why can a shipment have a carrier but no tracking number?

Because the carrier can be assigned at the beginning of the order, while the tracking number is generated later in the operational process.

In that case, the order still appears in the `guias` array with `guia: "Sin guia"` and `trackingUrl: null`:

```json
{
  "pedidoId": "25086",
  "paqueteria": "FLETERA",
  "guias": [
    {
      "guia": "Sin guia",
      "trackingUrl": null
    }
  ],
  "estatusEnvio": "activo",
  "fechaPedido": "2025-09-18T12:35:39.553"
}
```

### Why does `/api/envios/{folio}` return an array?

A single folio can be split into multiple internal orders. The API returns one object per `pedidoId` so integrations can show each internal order separately.

If one internal order has multiple guides, those guides are grouped inside the same `guias` array instead of repeating the order.

### What does the stock breakdown in a pre-order item mean?

Each pre-order item is enriched with the deliverable stock (`deliverable_qty`) of every **sales warehouse** (`sales_enabled = 1`). From that, the API computes how much of the requested quantity can be fulfilled (`cantidadCubierta` / `coveredQuantity`), how much is missing (`cantidadAgotada` / `shortageQuantity`), a suggested per-warehouse allocation, and a fulfillment status:

- `CUBIERTA` / `COVERED` — a single warehouse can cover the whole quantity.
- `DISTRIBUIR` / `DISTRIBUTE` — there is enough total stock, but it is spread across several warehouses.
- `AGOTADO_PARCIAL` / `PARTIALLY_COVERED` — there is some stock, but not enough for the full quantity.
- `SIN_STOCK` / `OUT_OF_STOCK` — no sales warehouse holds any stock.

### Why is `/api/preorders/detail` in English while the other pre-order endpoints are in Spanish?

That endpoint (route, JSON field names, and status values) is kept in English because it is reviewed by the China team. It returns the same information as `GET /api/PreOrdenes/{id}`, only translated. The other endpoints keep their original Spanish fields for backward compatibility.

The sales API (`/api/sales`) follows the same rule: it is fully in English because it is handled by the China team.

### Why does a sale have one status and its warehouses another?

Because of how the operational process works. A sale is stored in `quotation` with an **overall status**; while it is still a quotation that is the only status there is, and `isQuotation` comes back as `true` with an empty `warehouses[]`.

Once the quotation is validated, the order is **split and each warehouse keeps its own status**: one warehouse can already be printing shipping labels (`IN_SHIPPING_LABEL`) while another is still packing (`IN_PACKING_OR_REVIEW`). From that point on, `isQuotation` is `false` and the current status of each warehouse is in `warehouses[]`, plus in every item through `warehouseStatus`.

### What is the difference between `status` and `statusRaw` in the sales API?

`statusRaw` is the status name exactly as BambooERP stores it (in Spanish, for example `Pago Validado`), useful to trace the original value. `status` is that same value normalized to the English stages published by the API: `IN_QUOTATION`, `SENT_TO_CEDIS`, `IN_PICKING`, `IN_PACKING_OR_REVIEW`, `IN_SHIPPING_LABEL`, `DELIVERED`, `CANCELLED` or `UNKNOWN`.

Integrations should rely on `status`; `statusRaw` is informational and can add new values as BambooERP evolves.

### Why does `paymentsTotal` not match the sale `total`?

Because they answer different questions. `total` is what the sale closes with (column `quotation.total`), while `paymentsTotal` is the sum of the payments actually registered against it (`payments[].amount`).

They can legitimately differ in both directions: a sale can be partially paid (total 36,750 with 36,250.40 registered), or carry payments above its total. **Do not use `paymentsTotal` to decide that a sale is settled** — check the individual payments and their `status`.

### Can a sale have several payments with different payment forms?

Yes, and it is common. A single sale can combine a bank transfer, several credit-note balances, and cash. Each entry in `payments[]` carries its own amount, form (`paymentFormCode` / `paymentForm`), and status, so the payment mix can be reconstructed by grouping on the form.

The `paymentFormCode` is the SAT code from `sat_FormaPago`: `01` cash, `02` cheque, `03` electronic transfer, `04` credit card, `17` credit balance, `28` debit card, among others.

### Why do some sale items have no warehouse?

Because they are service lines (`isService: true`), such as `ENVIO %` or the freight carrier. They are not fulfilled by any warehouse, so they come back with `warehouseId: null` and `warehouseStatus: null`, and they are summed in `servicesTotal` instead of `productsSubtotal`.

### How do I get the payments that were rejected, validated, or are still being validated?

With `GET /payments` and the status in English: `status=REJECTED`, `status=VALID`, `status=IN_PROCESS` — or several at once, comma separated (`status=REJECTED,IN_PROCESS`). Omitting `status` returns every payment.

The response carries the full record of each payment plus a `summary` with the count and amount of each status **across the whole filter**, not just the page being read. A status outside the published list returns `400` instead of an empty page, so a typo is never read as "there are none".

### Why do `PENDING` and `IN_PROCESS` payments add up to zero?

Because the amount is not part of registering a payment: the ERP fills `amount` when someone validates it. A payment that is still pending or being validated exists, has a folio and a customer, and has no amount yet — so it counts in `summary` but adds `0.00`.

Only `VALID` and `REJECTED` payments carry amounts. Use `count` rather than `amount` to size those two statuses.
