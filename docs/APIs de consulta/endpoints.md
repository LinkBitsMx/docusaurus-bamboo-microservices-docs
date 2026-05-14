---
sidebar_position: 3
title: Endpoints
---

# Endpoints

## 1. Order query

Gets general order information.

```http
GET https://bamboonetapi.ddns.net/api/pedidos/{folio}
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
GET https://bamboonetapi.ddns.net/api/pedidos/{folio}/estatus
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

Gets the carrier, tracking number, and tracking URL associated with an order.

```http
GET https://bamboonetapi.ddns.net/api/envios/{folio}
```

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `folio` | string | Yes | Order folio. Example: `2505-00063` |

:::note
Due to the operational process, the carrier may be available before a tracking number exists. In that case, the API returns the carrier, `guia: "Sin guia"`, and `trackingUrl: null`.
:::

### Response with tracking number

```json
{
  "pedidoId": "78701",
  "paqueteria": "PAQUETEXPRESS",
  "guia": "MEX14PP0067946003003",
  "trackingUrl": "https://www.paquetexpress.com.mx/rastreo/MEX14PP0067946003003",
  "estatusEnvio": "activo",
  "fechaPedido": "2026-04-03T10:51:46.06"
}
```

### Response without tracking number

```json
{
  "pedidoId": "83",
  "paqueteria": "PAQUETEXPRESS",
  "guia": "Sin guia",
  "trackingUrl": null,
  "estatusEnvio": "activo",
  "fechaPedido": "2025-05-05T11:14:27.363"
}
```

## 4. Product prices

Gets prices by internal code or SKU.

```http
GET https://bamboonetapi.ddns.net/api/precios/productos/{identificador}
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
GET https://bamboonetapi.ddns.net/api/garantias/{folioTicket}
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
