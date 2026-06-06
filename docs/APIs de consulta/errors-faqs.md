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
