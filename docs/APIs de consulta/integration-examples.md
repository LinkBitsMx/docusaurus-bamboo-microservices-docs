---
sidebar_position: 4
title: Integration examples
---

# Integration examples

## Query shipment from JavaScript

```javascript
async function queryShipment(folio) {
  const response = await fetch(
    `http://pfconexionlinkbits.ddns.net:50780/api/envios/${encodeURIComponent(folio)}`,
    {
      method: 'GET',
      headers: {
        'X-API-Key': 'YOUR_API_KEY_HERE',
        'Accept': 'application/json',
      },
    }
  );

  if (response.status === 404) {
    return {
      found: false,
      message: 'No information was found for that folio.',
    };
  }

  if (!response.ok) {
    throw new Error(`HTTP error ${response.status}`);
  }

  const pedidos = await response.json();

  return {
    found: true,
    pedidos: pedidos.map((pedido) => ({
      pedidoId: pedido.pedidoId,
      paqueteria: pedido.paqueteria,
      estatusEnvio: pedido.estatusEnvio,
      fechaPedido: pedido.fechaPedido,
      guias: pedido.guias,
    })),
  };
}
```

## Suggested message for the end user

When the folio contains multiple internal orders:

```text
Shipment information found for folio 2509-03200:

Order 25086
Carrier: FLETERA
Tracking number: Sin guia

Order 25087
Carrier: PAQUETEXPRESS
Tracking numbers:
- MEX01PP3469501006006
- MEX01PP3469501006005
- MEX01PP3469501006004
- MEX01PP3469501006003
- MEX01PP3469501006002
- MEX01PP3469501006001
```

When one of the orders has tracking URLs:

```text
Your order already has a tracking number.
Carrier: PAQUETEXPRESS
Tracking number: MEX01PP3469501006006
Tracking URL: https://www.paquetexpress.com.mx/rastreo/MEX01PP3469501006006
```

When one of the orders has a carrier without a tracking number:

```text
Your order already has an assigned carrier: FLETERA.
The tracking number is not available yet. Please check again later.
```

When no information exists:

```text
No information was found for that folio. Please verify that it is written correctly.
```

## Query a sale with its per-warehouse status from JavaScript

```javascript
async function querySale(folio) {
  const response = await fetch(
    `http://pfconexionlinkbits.ddns.net:50780/api/sales/${encodeURIComponent(folio)}`,
    {
      method: 'GET',
      headers: {
        'X-API-Key': 'YOUR_API_KEY_HERE',
        'Accept': 'application/json',
      },
    }
  );

  if (response.status === 404) {
    return {
      found: false,
      message: 'No sale was found for that folio.',
    };
  }

  if (!response.ok) {
    throw new Error(`HTTP error ${response.status}`);
  }

  const sale = await response.json();

  return {
    found: true,
    folio: sale.folio,
    total: sale.totals.total,
    // While the sale is a quotation there is a single status and no warehouses yet.
    isQuotation: sale.status.isQuotation,
    status: sale.status.status,
    warehouses: sale.warehouses.map((warehouse) => ({
      warehouse: warehouse.warehouse,
      status: warehouse.status,
      units: warehouse.units,
      amount: warehouse.amount,
    })),
  };
}
```

To list the sales of a period, use `/api/sales` with `startDate`, `endDate`, and paging:

```javascript
async function listSales({ startDate, endDate, page = 1, pageSize = 50 }) {
  const query = new URLSearchParams({ startDate, endDate, page, pageSize });

  const response = await fetch(
    `http://pfconexionlinkbits.ddns.net:50780/api/sales?${query}`,
    {
      method: 'GET',
      headers: {
        'X-API-Key': 'YOUR_API_KEY_HERE',
        'Accept': 'application/json',
      },
    }
  );

  if (!response.ok) {
    throw new Error(`HTTP error ${response.status}`);
  }

  // totalPages tells you whether another page needs to be requested.
  return response.json();
}
```

## Suggested message for a split sale

```text
Sale 2607-00037 — total $130,997.00

Cedis Vallejo: packing in process (5,400 units)
Cedis Motevideo: shipping label in process (2,200 units)
```

When the sale is still a quotation:

```text
Sale 2608-00004 is still in quotation.
It has not been assigned to a warehouse yet.
```

## Recommended error handling

| Code | Suggested handling |
| --- | --- |
| `401` | Check the API Key. Do not show technical details to the end user. |
| `404` | Indicate that the folio was not found. |
| `500` | Ask the user to try again later or create an internal report. |
