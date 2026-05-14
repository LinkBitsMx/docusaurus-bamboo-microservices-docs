---
sidebar_position: 1
title: Introduction
slug: /query-apis
---

# Query APIs

This section documents the query APIs available for external integrations.

These APIs let external systems query BambooERP information without accessing the internal system. They are intended for scenarios where a bot, integration, or external channel needs to answer common questions about orders, shipments, prices, and warranties.

## What you can query

| API | What it solves |
| --- | --- |
| Order query | General order information by folio |
| Order status | Current progress of the order in the operational process |
| Shipment query | Carrier, tracking number, and tracking URL when available |
| Product prices | Price by internal code or SKU, separated by branch |
| Warranty query | Products and status associated with a warranty ticket folio |

## Base URL

```text
https://bamboonetapi.ddns.net/api/
```

## General format

- All requests use `GET`.
- All responses are returned as JSON.
- All requests require the `X-API-Key` header.
- If a value is not available yet in BambooERP, the API may return values such as `Sin guia`, `No asignada`, or `null`.

## Recommended flow

1. Request your API Key from the integration team.
2. Configure the API Key in your backend or private service.
3. Test first with real validation folios.
4. Handle `401`, `404`, and `500` responses correctly.
5. Do not expose the API Key in public frontend code, repositories, or screenshots.

## Endpoints

```text
GET /pedidos/{folio}
GET /pedidos/{folio}/estatus
GET /envios/{folio}
GET /precios/productos/{identificador}
GET /garantias/{folioTicket}
```
