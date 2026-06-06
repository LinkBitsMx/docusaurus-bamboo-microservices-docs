---
sidebar_position: 3
title: Endpoints
---

# Endpoints

## 1. Consulta de pedido

Obtiene la informacion general de un pedido.

```http
GET https://bamboonetapi.ddns.net/api/pedidos/{folio}
```

| Parametro | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `folio` | string | Si | Folio del pedido. Ejemplo: `2605-00005` |

### Respuesta

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

## 2. Estatus de pedido

Obtiene el avance actual del pedido.

```http
GET https://bamboonetapi.ddns.net/api/pedidos/{folio}/estatus
```

| Parametro | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `folio` | string | Si | Folio del pedido. Ejemplo: `2605-00005` |

```json
{
  "pedidoId": 78832,
  "estatus": "En proceso de surtido",
  "fechaEstatus": "2026-05-08T16:27:54.58"
}
```

## 3. Consulta de envio

Obtiene la informacion de envio asociada a un folio de pedido. Un mismo folio puede dividirse en mas de un pedido interno, por eso la respuesta es un arreglo agrupado por `pedidoId`.

```http
GET https://bamboonetapi.ddns.net/api/envios/{folio}
```

| Parametro | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `folio` | string | Si | Folio del pedido. Ejemplo: `2509-03200` |

:::note
Cada elemento de la respuesta representa un pedido interno (`pedidoId`). El arreglo `guias` contiene todas las guias y URLs de rastreo asociadas a ese pedido.

Debido al proceso operativo, una paqueteria puede estar disponible antes de que exista una guia. En ese caso el pedido sigue apareciendo en la respuesta con `guia: "Sin guia"` y `trackingUrl: null`.
:::

### Respuesta

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

## 4. Precios de producto

Obtiene precios por codigo interno o SKU.

```http
GET https://bamboonetapi.ddns.net/api/precios/productos/{identificador}
```

| Parametro | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `identificador` | string | Si | Codigo interno o SKU. Ejemplos: `000001`, `FDA07` |

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

## 5. Consulta de garantia

Obtiene los productos asociados a un folio de garantia.

```http
GET https://bamboonetapi.ddns.net/api/garantias/{folioTicket}
```

| Parametro | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `folioTicket` | string | Si | Folio del ticket de garantia |

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
