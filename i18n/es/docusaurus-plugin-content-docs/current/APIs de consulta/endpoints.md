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

Obtiene la paqueteria, guia y URL de rastreo asociadas a un pedido.

```http
GET https://bamboonetapi.ddns.net/api/envios/{folio}
```

| Parametro | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `folio` | string | Si | Folio del pedido. Ejemplo: `2505-00063` |

:::note
Debido al proceso operativo, la paqueteria puede estar disponible antes de que exista una guia. En ese caso la API regresa la paqueteria, `guia: "Sin guia"` y `trackingUrl: null`.
:::

### Respuesta con guia

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

### Respuesta sin guia

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
