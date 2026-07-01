---
sidebar_position: 3
title: Endpoints
---

# Endpoints

## 1. Order query

Gets general order information.

```http
GET http://pfconexionlinkbits.ddns.net:50780/api/pedidos/{folio}
```

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `folio` | string | Yes | Order folio. Example: `2605-00005` |

### Response

```json
{
  "pedidoId": 78832,
  "folio": "2605-00005",
  "cliente": "JESUS ADIEL DOMINGO MONSIVAIS",
  "fecha": "2026-05-08T16:27:21.947",
  "total": 450.0,
  "estatus": "ACTIVO"
}
```

## 2. Order status

Gets the current progress of the order.

```http
GET http://pfconexionlinkbits.ddns.net:50780/api/pedidos/{folio}/estatus
```

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `folio` | string | Yes | Order folio. Example: `2605-00005` |

```json
{
  "pedidoId": 78832,
  "estatus": "En proceso de surtido",
  "fechaEstatus": "2026-05-08T16:27:54.58"
}
```

## 3. Shipment query

Gets the shipment information associated with an order folio. A single folio can be split into more than one internal order, so the response is an array grouped by `pedidoId`.

```http
GET http://pfconexionlinkbits.ddns.net:50780/api/envios/{folio}
```

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `folio` | string | Yes | Order folio. Example: `2509-03200` |

:::note
Each item in the response represents one internal order (`pedidoId`). The `guias` array contains every tracking number and URL associated with that order.

Due to the operational process, a carrier may be available before a tracking number exists. In that case, the order still appears in the response with `guia: "Sin guia"` and `trackingUrl: null`.
:::

### Response

```json
[
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
  },
  {
    "pedidoId": "25087",
    "paqueteria": "PAQUETEXPRESS",
    "guias": [
      {
        "guia": "MEX01PP3469501006006",
        "trackingUrl": "https://www.paquetexpress.com.mx/rastreo/MEX01PP3469501006006"
      },
      {
        "guia": "MEX01PP3469501006005",
        "trackingUrl": "https://www.paquetexpress.com.mx/rastreo/MEX01PP3469501006005"
      },
      {
        "guia": "MEX01PP3469501006004",
        "trackingUrl": "https://www.paquetexpress.com.mx/rastreo/MEX01PP3469501006004"
      },
      {
        "guia": "MEX01PP3469501006003",
        "trackingUrl": "https://www.paquetexpress.com.mx/rastreo/MEX01PP3469501006003"
      },
      {
        "guia": "MEX01PP3469501006002",
        "trackingUrl": "https://www.paquetexpress.com.mx/rastreo/MEX01PP3469501006002"
      },
      {
        "guia": "MEX01PP3469501006001",
        "trackingUrl": "https://www.paquetexpress.com.mx/rastreo/MEX01PP3469501006001"
      }
    ],
    "estatusEnvio": "activo",
    "fechaPedido": "2025-09-18T12:35:39.557"
  }
]
```

## 4. Product prices

Gets prices by internal code or SKU.

```http
GET http://pfconexionlinkbits.ddns.net:50780/api/precios/productos/{identificador}
```

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `identificador` | string | Yes | Internal code or SKU. Examples: `000001`, `FDA07` |

```json
{
  "productoId": "555302",
  "codigo": "000001",
  "nombre": "FREIDORA DE AIRE FDA07",
  "sku": "FDA07",
  "preciosPorSucursal": [
    {
      "sucursal": "Mexico",
      "precioMayoreo": 560.0,
      "precioCaja": 560.0,
      "moneda": "MXN",
      "incluyeIva": true
    },
    {
      "sucursal": "Monterrey",
      "precioMayoreo": 565.0,
      "precioCaja": 565.0,
      "moneda": "MXN",
      "incluyeIva": true
    }
  ]
}
```

## 5. Warranty query

Gets the products associated with a warranty ticket folio.

```http
GET http://pfconexionlinkbits.ddns.net:50780/api/garantias/{folioTicket}
```

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `folioTicket` | string | Yes | Warranty ticket folio |

```json
{
  "folioTicket": "CMIC1E101898-D20260401-ID8503",
  "productos": [
    {
      "producto": "BOCINA KTS-1853",
      "fechaIngreso": "2026-04-01T10:07:13.94",
      "resultado": "No reparado",
      "estatus": "finalizado"
    }
  ]
}
```

## 6. Pre-orders

Lets an external customer system submit a **pre-order** (an unconfirmed quotation request). It is stored with status `PENDIENTE` so a salesperson can later take it and turn it into a quotation. This resource also exposes read endpoints to list and review incoming pre-orders.

### 6.1 Create pre-order

Creates a pre-order in status `PENDIENTE`. The server computes each line `amount` (`quantity * unitPrice`), the order `total`, and a readable `folio` with the format `{customerCode}-{id:D5}` (for example `C00123-00012`).

```http
POST http://pfconexionlinkbits.ddns.net:50780/api/PreOrdenes
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `customerCode` | string | Yes | Customer code. Max 200 chars. |
| `email` | string | No | Contact email. Max 200 chars, must be a valid email. |
| `phone` | string | No | Contact phone. Max 50 chars. |
| `notes` | string | No | Free-text notes for the salesperson. |
| `items` | array | Yes | At least one item is required. |
| `items[].productCode` | string | Yes | Product code or SKU. Max 50 chars. |
| `items[].quantity` | integer | Yes | Must be greater than zero. |
| `items[].unitPrice` | decimal | Yes | Cannot be negative. |

Request body:

```json
{
  "customerCode": "C00123",
  "email": "cliente@example.com",
  "phone": "8112345678",
  "notes": "Entregar por la tarde",
  "items": [
    { "productCode": "FDA07", "quantity": 2, "unitPrice": 560.00 },
    { "productCode": "000001", "quantity": 1, "unitPrice": 450.00 }
  ]
}
```

Response — `201 Created`. Returns the pre-order as it was persisted, including the generated `id`, `folio`, computed totals, and `createdAt`.

```json
{
  "id": 12,
  "folio": "C00123-00012",
  "customerCode": "C00123",
  "email": "cliente@example.com",
  "phone": "8112345678",
  "notes": "Entregar por la tarde",
  "status": "PENDIENTE",
  "isApproved": false,
  "total": 1570.00,
  "createdAt": "2026-07-01T10:15:30.123",
  "items": [
    { "id": 45, "productCode": "FDA07", "quantity": 2, "unitPrice": 560.00, "amount": 1120.00 },
    { "id": 46, "productCode": "000001", "quantity": 1, "unitPrice": 450.00, "amount": 450.00 }
  ]
}
```

### 6.2 List pre-orders

Lists pre-orders, optionally filtered by status. Results are ordered by `createdAt` descending.

```http
GET http://pfconexionlinkbits.ddns.net:50780/api/PreOrdenes?status={status}
```

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | string | No | Filter by status (case-insensitive). Example: `PENDIENTE`. If omitted, returns all pre-orders. |

### Response

```json
[
  {
    "id": 12,
    "folio": "C00123-00012",
    "customerCode": "C00123",
    "status": "PENDIENTE",
    "total": 1570.00,
    "totalItems": 2,
    "createdAt": "2026-07-01T10:15:30.123"
  }
]
```

### 6.3 Get pre-order by id

Gets the full detail of a pre-order, including its items. Returns `404` if no pre-order exists with the given `id`.

```http
GET http://pfconexionlinkbits.ddns.net:50780/api/PreOrdenes/{id}
```

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | Yes | Pre-order id. |

### Response

```json
{
  "id": 12,
  "folio": "C00123-00012",
  "customerCode": "C00123",
  "email": "cliente@example.com",
  "phone": "8112345678",
  "notes": "Entregar por la tarde",
  "status": "PENDIENTE",
  "isApproved": false,
  "total": 1570.00,
  "createdAt": "2026-07-01T10:15:30.123",
  "items": [
    { "id": 45, "productCode": "FDA07", "quantity": 2, "unitPrice": 560.00, "amount": 1120.00 },
    { "id": 46, "productCode": "000001", "quantity": 1, "unitPrice": 450.00, "amount": 450.00 }
  ]
}
```
