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
