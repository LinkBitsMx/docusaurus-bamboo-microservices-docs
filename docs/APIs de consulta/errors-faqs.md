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
