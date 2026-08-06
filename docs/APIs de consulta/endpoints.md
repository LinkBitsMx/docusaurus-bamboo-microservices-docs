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

Besides the base fields, **each item includes the stock breakdown per sales warehouse** (warehouses with `sales_enabled = 1`), using the deliverable stock (`deliverable_qty`). This mirrors the "Almacén a surtir (stock)" view of the quotation screen.

```http
GET http://pfconexionlinkbits.ddns.net:50780/api/PreOrdenes/{id}
```

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | Yes | Pre-order id. |

Per-item stock fields (Spanish, same language as the rest of this endpoint):

| Field | Type | Description |
| --- | --- | --- |
| `stockDisponible` | integer | Total deliverable stock summed across all sales warehouses. |
| `cantidadCubierta` | integer | How much of the requested quantity can be fulfilled with the available stock. |
| `cantidadAgotada` | integer | How much cannot be fulfilled (shortage / out of stock). |
| `estadoSurtido` | string | `CUBIERTA` (a single warehouse covers it all), `DISTRIBUIR` (enough stock but spread across warehouses), `AGOTADO_PARCIAL` (stock is short) or `SIN_STOCK` (no warehouse has stock). |
| `almacenes[]` | array | Only warehouses that hold stock, ordered from most to least. |
| `almacenes[].almacenId` | integer | Warehouse id. |
| `almacenes[].almacen` | string | Warehouse name. |
| `almacenes[].stockDisponible` | integer | Deliverable units in this warehouse. |
| `almacenes[].cantidadSurtir` | integer | Units suggested to fulfill from this warehouse (greedy allocation). |

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
    {
      "id": 45,
      "productCode": "FDA07",
      "quantity": 2,
      "unitPrice": 560.00,
      "amount": 1120.00,
      "stockDisponible": 1236,
      "cantidadCubierta": 2,
      "cantidadAgotada": 0,
      "estadoSurtido": "CUBIERTA",
      "almacenes": [
        { "almacenId": 1540424, "almacen": "Almacen San Martin", "stockDisponible": 970, "cantidadSurtir": 2 },
        { "almacenId": 1540415, "almacen": "Almacen Apodaca",     "stockDisponible": 266, "cantidadSurtir": 0 }
      ]
    },
    {
      "id": 46,
      "productCode": "000001",
      "quantity": 1,
      "unitPrice": 450.00,
      "amount": 450.00,
      "stockDisponible": 0,
      "cantidadCubierta": 0,
      "cantidadAgotada": 1,
      "estadoSurtido": "SIN_STOCK",
      "almacenes": []
    }
  ]
}
```

### 6.4 List pre-orders with detail (English endpoint)

Lists **every** pre-order (optionally filtered by status) with the same enriched per-item detail as `6.3` (stock per warehouse, covered/shortage quantity, fulfillment status and suggested allocation). The stock for all items is resolved in a single query.

:::info Language
This endpoint is fully **in English** — route, JSON field names, and status values — because it is reviewed by the China team. The other pre-order endpoints stay in Spanish. It is served on an absolute route (`/api/preorders/detail`), outside the `/api/PreOrdenes` prefix.
:::

```http
GET http://pfconexionlinkbits.ddns.net:50780/api/preorders/detail?status={status}
```

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | string | No | Filter by status in **English**: `PENDING`, `TAKEN`, `CONVERTED`, `CANCELLED`. If omitted, returns all pre-orders. |

Per-item fields (English equivalents of the Spanish fields in `6.3`):

| Field | Type | Description |
| --- | --- | --- |
| `availableStock` | integer | Total deliverable stock across all sales warehouses. |
| `coveredQuantity` | integer | How much of the requested quantity can be fulfilled. |
| `shortageQuantity` | integer | How much cannot be fulfilled (shortage). |
| `fulfillmentStatus` | string | `COVERED`, `DISTRIBUTE`, `PARTIALLY_COVERED` or `OUT_OF_STOCK`. |
| `warehouses[]` | array | Only warehouses that hold stock, ordered from most to least. |
| `warehouses[].warehouseId` | integer | Warehouse id. |
| `warehouses[].warehouse` | string | Warehouse name. |
| `warehouses[].availableStock` | integer | Deliverable units in this warehouse. |
| `warehouses[].quantityToFulfill` | integer | Units suggested to fulfill from this warehouse (greedy allocation). |

The pre-order `status` is also returned in English: `PENDING`, `TAKEN`, `CONVERTED`, `CANCELLED`.

### Response

```json
[
  {
    "id": 21,
    "folio": "NLN2A101766-00021",
    "customerCode": "NLN2A101766",
    "email": null,
    "phone": null,
    "notes": null,
    "status": "PENDING",
    "isApproved": false,
    "total": 153190.00,
    "createdAt": "2026-07-27T09:40:00.000",
    "items": [
      {
        "id": 84,
        "productCode": "001135",
        "quantity": 40,
        "unitPrice": 929.00,
        "amount": 37160.00,
        "availableStock": 1710,
        "coveredQuantity": 40,
        "shortageQuantity": 0,
        "fulfillmentStatus": "COVERED",
        "warehouses": [
          { "warehouseId": 1540424, "warehouse": "Almacen San Martin", "availableStock": 819, "quantityToFulfill": 40 },
          { "warehouseId": 1540418, "warehouse": "Cedis Vallejo",       "availableStock": 283, "quantityToFulfill": 0 }
        ]
      }
    ]
  }
]
```

## 7. Sales (English endpoint)

Detailed query of the sales registered in BambooERP: header, customer, totals, invoicing, the line-by-line detail and the status of each warehouse.

:::info Language
This API is fully **in English** — routes, JSON field names, and status values — because it is handled by the China team, same as `6.4`. The other endpoints stay in Spanish.
:::

### How a sale is modelled

A sale is stored in `quotation` (header, totals and **overall status**) and its detail in `quotation_detail`, where every line holds the warehouse that fulfills it.

- While the sale is a **quotation**, that overall status is the only one that exists: `isQuotation` is `true` and `warehouses[]` comes back empty.
- Once the quotation is validated, **the order is split and each warehouse keeps its own status**. At that point `isQuotation` turns `false` and `warehouses[]` carries the current status of each one.

In practice, sales not split yet sit in status `Sin procesar` and split ones in `Pago Validado`, but `isQuotation` is computed from the actual existence of per-warehouse orders, not from the status name.

Every status is returned twice:

| Field | Description |
| --- | --- |
| `statusRaw` | Status name exactly as stored in BambooERP (in Spanish). Useful to trace the original value. |
| `status` | That same value normalized to the English stages below. |

**Possible values of `status`:**

| Value | Meaning |
| --- | --- |
| `IN_QUOTATION` | Still being quoted |
| `SENT_TO_CEDIS` | Payment processed, sent to the CEDIS |
| `IN_PICKING` | Being picked |
| `IN_PACKING_OR_REVIEW` | Being packed or reviewed |
| `IN_SHIPPING_LABEL` | Shipping label in process |
| `DELIVERED` | Collected or delivered |
| `CANCELLED` | Cancelled |
| `UNKNOWN` | Status not mapped |

### Branch and department

A sale reaches its branch through its department:

```text
quotation.DepartamentoId -> departments.id
departments.branchId     -> starnet_branches.id
```

Filter with `branchCode` in `7.1`, using `starnet_branches.code` (for example `801.10.02` for Sucursal Florida). One branch groups several departments, so `801.01.01` (REGIONES) covers the sales of Oficina, Rutas, Region Noreste, and AEK together.

The list returns `branchCode` and `branch` (name). The detail returns a `branch` object and the `department` it came from:

| Field | Type | Description |
| --- | --- | --- |
| `branch.branchId` | integer | Branch id (`starnet_branches.id`). |
| `branch.code` | string | Branch code — the value the `branchCode` filter takes. Example: `801.10.02` |
| `branch.name` | string | Branch name. Example: `Sucursal Florida` |
| `department.departmentId` | integer | Department id (`departments.id`). |
| `department.code` | string | Department code. Example: `801010302` |
| `department.name` | string | Department name. |
| `department.zone` | string | Zone it belongs to. Example: `CDMX`, `GDL`, `MTY` |

:::note
A few sales do not resolve a branch: either they have no department, or they point to a department id that no longer exists in `departments`. In those cases `branch` and `department` come back as `null`.
:::

### Seller and payments

The salesperson who registered the sale comes from `quotation.usuarioId` joined against `catUsers`. The list returns `sellerId` and `seller` (full name); the detail returns a `seller` object with code, email, and username.

The detail also returns `payments[]`, the payments registered against the sale with their payment form. The relation goes:

```text
quotation.id                     -> rel_quotes_to_payments.Quote_id
rel_quotes_to_payments.VoucherId -> Payments.Id
```

:::note
The payment is carried by `VoucherId`, not by `payment_id`. Rows with `VoucherId = 0` are placeholders without a payment and are skipped.
:::

| Field | Type | Description |
| --- | --- | --- |
| `payments[].paymentId` | integer | Payment id (`Payments.Id`). |
| `payments[].folio` | string | Payment folio. Example: `PAY-0726-000010` |
| `payments[].paymentDate` | datetime | Date of the payment. |
| `payments[].amount` | decimal | Amount of this payment. |
| `payments[].paymentFormCode` | string | SAT code of the payment form (`sat_FormaPago.vchCode`). Example: `03` |
| `payments[].paymentForm` | string | Payment form name. Example: `Transferencia electronica de fondos` |
| `payments[].statusRaw` | string | Payment status as stored in BambooERP (in Spanish). |
| `payments[].status` | string | `VALID`, `REJECTED`, `PENDING`, `IN_PROCESS`, `CANCELLED` or `UNKNOWN`. |
| `payments[].reference` | string | Bank reference, when registered. |
| `payments[].paymentType` | string | Payment type as stored. Example: `payment` |
| `payments[].createdAt` | datetime | When the payment was tied to the sale. |

A sale can carry several payments with different forms — for example a transfer, a set of credit-note balances, and cash on the same sale. `totals.paymentsTotal` sums `payments[].amount`.

:::warning
`paymentsTotal` does **not** have to match `total`: a sale can be partially paid, or carry payments registered above its total. Do not use it to infer that a sale is settled.
:::

### 7.1 List sales

Paged list of sales, ordered from newest to oldest. Every sale carries its overall status and, once the quotation has been validated, the status of each warehouse order.

```http
GET http://pfconexionlinkbits.ddns.net:50780/api/sales
```

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `startDate` | date | No | Lower bound of the sale date. Example: `2026-07-01` |
| `endDate` | date | No | Upper bound of the sale date. If sent without a time part, the whole day is included. |
| `customerCode` | string | No | Exact customer code. Example: `SLP2A101255` |
| `folio` | string | No | Partial match on the folio. Example: `2607-` |
| `statusId` | integer | No | **Overall** status of the sale (`quotation.status_id`). Only 5 values ever reach it — see the warning below. |
| `warehouseStatusId` | integer | No | Status of the **per-warehouse orders**: keeps sales where at least one warehouse is currently in that status. Example: `21` (Recolectado) |
| `warehouseId` | integer | No | Only sales with lines fulfilled by that warehouse. |
| `branchCode` | string | No | Branch code (`starnet_branches.code`). Example: `801.10.02` (Sucursal Florida) |
| `onlyQuotations` | boolean | No | `true` keeps only the sales still in quotation. |
| `includePayments` | boolean | No | `true` adds `payments[]` and `paymentsTotal` to every sale in the list. Default `false`. |
| `page` | integer | No | Page number. Default `1`. |
| `pageSize` | integer | No | Page size. Default `50`, maximum `200`. |

:::warning `statusId` is not the fulfillment stage
`quotation.status_id` only ever holds 5 values: `1` Sin procesar, `27` Pago Validado, `23` Cancelado, `28` Pago no valido and `29` Pago sin proceso.

The fulfillment stages — `21` Recolectado, `18` Guia Generada, `15` Empacado Finalizado, and the rest — live on the **per-warehouse orders**, so `statusId=21` always returns 0 sales. Use `warehouseStatusId=21` instead.
:::

By default the list does **not** include payments, because they cost one extra query per page. Ask for them with `includePayments=true` and every sale gains `payments[]` (same fields as the detail) plus `paymentsTotal`. The detail endpoint always returns them.

Example with filters:

```http
GET http://pfconexionlinkbits.ddns.net:50780/api/sales?startDate=2026-07-01&endDate=2026-07-31&pageSize=50
GET http://pfconexionlinkbits.ddns.net:50780/api/sales?branchCode=801.01.01&startDate=2026-05-01&endDate=2026-06-01&warehouseStatusId=21&includePayments=true
```

:::note
All filters travel in the **query string**. This is a `GET`: filters sent as a JSON body are ignored, and the request comes back as if it had no filters at all.
:::

### Response

```json
{
  "page": 1,
  "pageSize": 50,
  "totalRecords": 46,
  "totalPages": 1,
  "sales": [
    {
      "saleId": 78989,
      "folio": "2607-00037",
      "date": "2026-07-21T15:58:26.343",
      "customerCode": "SLP2A101255",
      "customer": "JESUS ADIEL DOMINGO MONSIVAIS",
      "branchCode": "801.10.02",
      "branch": "Sucursal Florida",
      "sellerId": 167,
      "seller": "Fernando Dominguez Garcia",
      "statusId": 27,
      "statusRaw": "Pago Validado",
      "status": "SENT_TO_CEDIS",
      "isQuotation": false,
      "units": 7600,
      "totalLines": 7,
      "total": 130997.0,
      "warehouses": [
        {
          "warehouseId": 1540420,
          "warehouse": "Cedis Motevideo",
          "statusId": 17,
          "statusRaw": "Guia en proceso",
          "status": "IN_SHIPPING_LABEL"
        },
        {
          "warehouseId": 1540418,
          "warehouse": "Cedis Vallejo",
          "statusId": 11,
          "statusRaw": "Empacado sin procesar",
          "status": "IN_PACKING_OR_REVIEW"
        }
      ]
    }
  ]
}
```

### 7.2 Sale detail

Gets the full detail of a sale by folio. Returns `404` if no sale exists with that folio.

On top of the header and the full detail, `warehouses[]` summarizes each per-warehouse order (status, units, lines and amount it accounts for) and every item states the warehouse fulfilling it along with the status of that order.

```http
GET http://pfconexionlinkbits.ddns.net:50780/api/sales/{folio}
```

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `folio` | string | Yes | Sale folio. Example: `2607-00037` |

Totals:

| Field | Type | Description |
| --- | --- | --- |
| `units` | integer | Units summed from the product lines. |
| `totalLines` | integer | Number of detail lines (products and services). |
| `productsSubtotal` | decimal | Sum of the product lines. |
| `servicesTotal` | decimal | Sum of the service lines (shipping, freight, etc.). |
| `lineDiscount` | decimal | Discount summed from the detail lines. |
| `paymentsTotal` | decimal | Sum of `payments[].amount` — what has actually been paid. |
| `deliveryTotal` | decimal | Column `total_deliver` of `quotation`. |
| `assuredTotal` | decimal | Column `total_of_assured` of `quotation`. |
| `freightCarrierTotal` | decimal | Column `total_fletera` of `quotation`. |
| `total` | decimal | The amount the sale closes with (column `total`). |
| `initialTotal` | decimal | Total before any modification (column `total_initial`). |
| `hasDiscount` | boolean | Whether the sale carries a discount. |

Per-warehouse fields:

| Field | Type | Description |
| --- | --- | --- |
| `warehouses[].warehouseId` | integer | Warehouse id. |
| `warehouses[].warehouse` | string | Warehouse name. |
| `warehouses[].statusRaw` | string | Status of that warehouse order as stored in BambooERP. |
| `warehouses[].status` | string | Same status normalized to English. |
| `warehouses[].assignedAt` | datetime | When the order was assigned to the warehouse. |
| `warehouses[].units` | integer | Units this warehouse fulfills. |
| `warehouses[].totalLines` | integer | Detail lines this warehouse fulfills. |
| `warehouses[].amount` | decimal | Amount those lines account for. |

Per-item fields:

| Field | Type | Description |
| --- | --- | --- |
| `items[].productCode` | string | Internal product code. |
| `items[].sku` | string | Product SKU. |
| `items[].product` | string | Product name. |
| `items[].quantity` | integer | Quantity ordered. |
| `items[].unitPrice` | decimal | Unit price. |
| `items[].discount` | decimal | Discount applied to the line. |
| `items[].amount` | decimal | Line amount (`quantity * unitPrice`). |
| `items[].isService` | boolean | `true` for service lines such as shipping or freight. |
| `items[].warehouseId` | integer | Warehouse fulfilling the line (`null` for services). |
| `items[].warehouse` | string | Warehouse name. |
| `items[].warehouseStatus` | string | Status of that warehouse order (`null` while still a quotation). |
| `items[].notes` | string | Line notes. |

### Response

```json
{
  "saleId": 78989,
  "folio": "2607-00037",
  "date": "2026-07-21T15:58:26.343",
  "customer": {
    "customerCode": "SLP2A101255",
    "name": "JESUS ADIEL DOMINGO MONSIVAIS",
    "email": "",
    "phone": "",
    "branch": null
  },
  "seller": {
    "sellerId": 167,
    "name": "Fernando Dominguez Garcia",
    "code": "testLuisilloPillo",
    "email": "Fernando.DominguezGarcia@gmail.com",
    "username": "1369"
  },
  "branch": {
    "branchId": 1148512,
    "code": "801.10.02",
    "name": "Sucursal Florida"
  },
  "department": {
    "departmentId": 10,
    "code": "801010302",
    "name": "Sucursal Florida",
    "zone": "CDMX"
  },
  "status": {
    "statusId": 27,
    "statusRaw": "Pago Validado",
    "status": "SENT_TO_CEDIS",
    "isQuotation": false,
    "statusDate": "2026-07-21T16:37:14.49"
  },
  "totals": {
    "units": 7600,
    "totalLines": 7,
    "productsSubtotal": 129700.0,
    "servicesTotal": 6127.0,
    "lineDiscount": 0.0,
    "paymentsTotal": 135827.0,
    "deliveryTotal": 0.0,
    "assuredTotal": 1297.0,
    "freightCarrierTotal": 0.0,
    "total": 130997.0,
    "initialTotal": null,
    "hasDiscount": false
  },
  "invoicing": {
    "requiresInvoice": false,
    "invoiced": false,
    "isCredit": false,
    "isDirectSale": false
  },
  "warehouses": [
    {
      "warehouseId": 1540418,
      "warehouse": "Cedis Vallejo",
      "statusId": 11,
      "statusRaw": "Empacado sin procesar",
      "status": "IN_PACKING_OR_REVIEW",
      "assignedAt": "2026-07-21T16:26:10.67",
      "units": 5400,
      "totalLines": 3,
      "amount": 34900.0
    },
    {
      "warehouseId": 1540420,
      "warehouse": "Cedis Motevideo",
      "statusId": 17,
      "statusRaw": "Guia en proceso",
      "status": "IN_SHIPPING_LABEL",
      "assignedAt": "2026-07-21T16:26:10.67",
      "units": 2200,
      "totalLines": 2,
      "amount": 94800.0
    }
  ],
  "items": [
    {
      "itemId": 711380,
      "productCode": "000449",
      "sku": "B03W10",
      "product": "FOCO LED B03W10",
      "quantity": 5000,
      "unitPrice": 5.0,
      "discount": 0.0,
      "amount": 25000.0,
      "isService": false,
      "warehouseId": 1540418,
      "warehouse": "Cedis Vallejo",
      "warehouseStatus": "IN_PACKING_OR_REVIEW",
      "notes": null
    },
    {
      "itemId": 711385,
      "productCode": "LB00001",
      "sku": null,
      "product": "ENVIO %",
      "quantity": 1,
      "unitPrice": 1297.0,
      "discount": 0.0,
      "amount": 1297.0,
      "isService": true,
      "warehouseId": null,
      "warehouse": null,
      "warehouseStatus": null,
      "notes": null
    }
  ],
  "payments": [
    {
      "paymentId": 34049,
      "folio": "PAY-0726-000010",
      "paymentDate": "2026-07-21T00:00:00",
      "amount": 135827.0,
      "paymentFormCode": "01",
      "paymentForm": "Efectivo",
      "statusId": 29,
      "statusRaw": "Valido",
      "status": "VALID",
      "reference": null,
      "paymentType": "payment",
      "createdAt": "2026-07-21T16:36:06.54"
    }
  ]
}
```

## 8. Payments (English endpoint)

Registers payments in BambooERP (`Payments`), the same table the ERP writes when a voucher is uploaded. It also exposes a read endpoint to get a payment back by id.

:::info Language
This API is fully **in English** — routes, JSON field names, and status values — because it is handled by the China team, same as `6.4` and `7`. The other endpoints stay in Spanish.
:::

### How a payment is registered

A payment is created **pending validation**: status `4` (PENDIENTE) and no amount yet. The amount and the final status (`29` Valido / `30` Rechazado) are set later by whoever validates it inside the ERP. You can still send `amount` at creation time, and `statusId` if you need to override the default.

Two things are done by the database itself, so they are **not** part of the request:

| Done by | What it does |
| --- | --- |
| Trigger `trg_GenerarFolioPayments` | Generates `folio` with the format `PAY-MMYY-NNNNNN`, a consecutive per month. That is why `folio` is not sent in the body. |
| Trigger `trg_AfterInsert_Payments_InsertRelation` | Relates the payment to the sale in `rel_quotes_to_payments` when `saleFolio` is sent. |

:::warning Sending `saleFolio` moves the sale
When the payment is related to a sale, the database also moves that sale to status `29`. Omit `saleFolio` if you only want to register the payment without touching the sale.
:::

The receiving company is not sent either: it is derived from the bank account.

```text
bankId -> bancos.id
bancos.id_origen -> origen_cuenta.id   (accountId in the response)
```

Every status is returned twice, same as in `7`: `statusRaw` is the value as stored in BambooERP (in Spanish) and `status` is that value normalized to `VALID`, `REJECTED`, `PENDING`, `IN_PROCESS`, `CANCELLED` or `UNKNOWN`.

### Kingdee fields

The payment can carry the fields of the Kingdee recharge document (充值单). Six of them are already covered by what the payment stores and must **not** be sent — they are derived from BambooERP catalogs. The rest are Kingdee's own identifiers: BambooERP has no catalog to resolve them against, so they are stored on `Payments` exactly as they arrive.

| Kingdee field | Body field | Where it comes from |
| --- | --- | --- |
| `FBillNo` (单据编号) | `kingdeeBillNo` | Sent. It does not replace `folio`, which the database keeps generating. |
| `FDate` (单据日期) | — | `paymentDate` |
| `FBizOrgId` / `FBizOrg` (业务组织) | `bizOrgId` / `bizOrgCode` | Sent |
| `FSETTLEORGID` / `FSETTLEORG` (结算组织) | `settleOrgId` / `settleOrgCode` | Sent |
| `FBranchID` / `Fbranch` (充值门店) | — | `departmentId` → `departments.branchId` → `starnet_branches.id` / `.code` |
| `FSalerID` / `FSaler` (业务员) | — | `sellerId` → `catUsers.code_seller`, stored as `kingdeeId_kingdeeCode_branchId` |
| `FCashierID` / `FCashier` (收银员) | `cashierId` / `cashierCode` | Sent. The cashier in Kingdee, which is not necessarily `uploadedById`. |
| `FCustomerID` / `FCustomer` (客户) | — | `customerCode` → `customers.customer_id` / `customer_code` |
| `FSETTLECURRENCYID` / `FSETTLECURRENCY` (结算币别) | `settleCurrencyId` / `settleCurrencyCode` | Sent |
| `FNote` (备注) | — | `comentary` |
| `FCardID` / `FCard` (卡号) | `cardId` / `cardNumber` | Sent |
| `FMemberID` / `FMember` (会员卡号) | `memberId` / `memberCardNumber` | Sent |
| `FAccountID` / `FAccount` (账户) | `kingdeeAccountId` / `kingdeeAccountCode` | Sent. Kingdee's account — unrelated to `accountId` in the response, which is the receiving company. |
| `FRechargeAmount` (充值金额) | `rechargeAmount` | Sent. What is credited to the card, as opposed to `amount`, which is what was collected. |
| `FReceiveTypeID` / `FReceiveType` (收款方式) | `receiveTypeId` / `receiveTypeCode` | Sent. Kingdee's receipt method, independent from `paymentFormId` (the SAT payment form). |
| `FReceiveCurrencyID` / `FReceiveCurrency` (收款币别) | `receiveCurrencyId` / `receiveCurrencyCode` | Sent. Defaults to the settlement currency. |
| `FReceiveAmt` (收款金额) | — | `amount` |
| `FExchangeRate` (汇率) | `exchangeRate` | Sent. Defaults to `1` while both currencies match; **required when they differ**, since BambooERP has no exchange rate table. |

All of them are optional, so the body the ERP already sends keeps working unchanged. The response carries a `kingdee` block with the document assembled and the `F*` names spelled exactly as Kingdee expects them (that block is the only part of the API that is not `camelCase`).

### 8.1 Register payment

```http
POST http://pfconexionlinkbits.ddns.net:50780/api/payments
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `customerCode` | string | Yes | Customer the payment belongs to (`customers.customer_code`). Max 50 chars. |
| `bankId` | integer | Yes | Bank account the money was deposited into (`bancos.id`). Must exist and not be disabled. |
| `paymentFormId` | integer | Yes | Payment form (`sat_FormaPago.ID`). Example: `3` = transferencia. |
| `uploadedById` | integer | Yes | User registering the payment (`catUsers.idUsuario`). |
| `paymentDate` | date | No | Date of the payment. Defaults to today. |
| `amount` | decimal | No | Amount. Must be greater than zero when sent. Left empty until validation, same as the ERP. |
| `reference` | string | No | Bank reference of the transfer or deposit. Max 250 chars. |
| `paymentType` | string | No | `payment` (default), `credit` or `advance`. |
| `paymentFilePath` | string | No | Path of the uploaded voucher file. Max 250 chars. |
| `saleFolio` | string | No | Folio of the sale the payment is applied to (`quotation.billCode`). See the warning above. |
| `sellerId` | integer | No | Salesperson the payment is credited to (`catUsers.idUsuario`). |
| `departmentId` | integer | No | Branch the payment belongs to (`departments.id`). Defaults to the branch of `uploadedById`. |
| `statusId` | integer | No | Status the payment is created with (`catEstatus.idEstatus`). Defaults to `4` (PENDIENTE). |
| `comentary` | string | No | Comment. Max 1500 chars. |
| `observations` | string | No | Observations. Max 500 chars. |

Plus the Kingdee fields described above, all optional: `kingdeeBillNo`, `bizOrgId`, `bizOrgCode`, `settleOrgId`, `settleOrgCode`, `cashierId`, `cashierCode`, `kingdeeAccountId`, `kingdeeAccountCode`, `receiveTypeId`, `receiveTypeCode`, `settleCurrencyId`, `settleCurrencyCode`, `receiveCurrencyId`, `receiveCurrencyCode`, `exchangeRate`, `cardId`, `cardNumber`, `memberId`, `memberCardNumber` and `rechargeAmount`.

Request body:

```json
{
  "customerCode": "SIN2A100652",
  "paymentDate": "2026-08-05",
  "bankId": 7,
  "paymentFormId": 3,
  "amount": 504.70,
  "reference": "0123456789",
  "paymentType": "payment",
  "paymentFilePath": "comprobante-1785955090.jpeg",
  "saleFolio": "2608-00012",
  "uploadedById": 426,
  "sellerId": 123,
  "comentary": "Transferencia recibida",

  "kingdeeBillNo": "SKCZD000123",
  "bizOrgId": 847244,
  "bizOrgCode": "801",
  "settleOrgId": 847244,
  "settleOrgCode": "801",
  "cashierId": 1772,
  "cashierCode": "GW000041",
  "kingdeeAccountId": 100012,
  "kingdeeAccountCode": "BANK001",
  "receiveTypeId": 5,
  "receiveTypeCode": "SKFS03",
  "settleCurrencyId": 1,
  "settleCurrencyCode": "MXN",
  "receiveCurrencyId": 1,
  "receiveCurrencyCode": "MXN",
  "exchangeRate": 1.0,
  "cardId": 5001,
  "cardNumber": "6234567890",
  "memberId": 8801,
  "memberCardNumber": "VIP00034",
  "rechargeAmount": 504.70
}
```

Response — `201 Created`. Returns the payment as it was persisted, including the `folio` generated by the database.

```json
{
  "paymentId": 34076,
  "folio": "PAY-0826-000024",
  "customerCode": "SIN2A100652",
  "customer": "BAUDELIO GONZALEZ VAZQUEZ",
  "paymentDate": "2026-08-05T00:00:00",
  "amount": 504.70,
  "paymentFormId": 3,
  "paymentFormCode": "03",
  "paymentForm": "Transferencia electronica de fondos",
  "accountId": 3,
  "account": "XIAN INTERNATIONAL SA DE CV",
  "bankId": 7,
  "bank": "BBVA",
  "bankAccountNumber": "0124482190",
  "statusId": 4,
  "statusRaw": "PENDIENTE",
  "status": "PENDING",
  "reference": "0123456789",
  "paymentType": "payment",
  "saleId": 79021,
  "saleFolio": "2608-00012",
  "departmentId": 16,
  "department": "Sucursal Ramon Corona",
  "uploadedById": 426,
  "sellerId": 123,
  "paymentFilePath": "comprobante-1785955090.jpeg",
  "comentary": "Transferencia recibida",
  "observations": null,
  "createdAt": "2026-08-05T13:03:30.61",
  "kingdee": {
    "FBillNo": "SKCZD000123",
    "FDate": "2026-08-05T00:00:00",
    "FBizOrgId": 847244,
    "FBizOrg": "801",
    "FSETTLEORGID": 847244,
    "FSETTLEORG": "801",
    "FBranchID": 1148514,
    "Fbranch": "801.10.04",
    "FSalerID": 1772,
    "FSaler": "GW000041",
    "FCashierID": 1772,
    "FCashier": "GW000041",
    "FCustomerID": 5966485,
    "FCustomer": "SIN2A100652",
    "FSETTLECURRENCYID": 1,
    "FSETTLECURRENCY": "MXN",
    "FNote": "Transferencia recibida",
    "FCardID": 5001,
    "FCard": "6234567890",
    "FMemberID": 8801,
    "FMember": "VIP00034",
    "FAccountID": 100012,
    "FAccount": "BANK001",
    "FRechargeAmount": 504.70,
    "FReceiveTypeID": 5,
    "FReceiveType": "SKFS03",
    "FReceiveCurrencyID": 1,
    "FReceiveCurrency": "MXN",
    "FReceiveAmt": 504.70,
    "FExchangeRate": 1.0
  }
}
```

Every reference is checked against BambooERP before the payment is inserted. If one does not exist, nothing is written and the API returns `400` with the reason:

```json
{ "message": "Customer 'NO-EXISTE' not found." }
```

| Case | Message |
| --- | --- |
| Customer not found | `Customer '{customerCode}' not found.` |
| Bank account not found | `Bank account {bankId} not found.` |
| Bank account disabled | `Bank account {bankId} is disabled.` |
| Payment form not found | `Payment form {paymentFormId} not found.` |
| User, seller, department or status not found | `{Entity} {id} not found.` |
| Sale folio not found | `Sale '{saleFolio}' not found.` |
| Invalid payment type | `paymentType must be one of: payment, credit, advance.` |
| Currencies differ without a rate | `exchangeRate is required when settleCurrencyCode and receiveCurrencyCode differ.` |

### 8.2 Get payment

Gets a payment by id, with the same structure as the response above. Returns `404` if no payment exists with that id.

```http
GET http://pfconexionlinkbits.ddns.net:50780/api/payments/{id}
```

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | Yes | Payment id (`Payments.Id`). Example: `34076` |
