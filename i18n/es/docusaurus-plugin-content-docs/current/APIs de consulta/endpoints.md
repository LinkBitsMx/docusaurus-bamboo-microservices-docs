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

## 6. Pre-ordenes

Permite que un sistema externo de clientes envie una **pre-orden** (una solicitud de cotizacion sin confirmar). Se guarda con estatus `PENDIENTE` para que un vendedor la tome despues y la convierta en cotizacion. Este recurso tambien expone endpoints de lectura para listar y revisar las pre-ordenes recibidas.

### 6.1 Crear pre-orden

Crea una pre-orden en estatus `PENDIENTE`. El servidor calcula el `amount` de cada renglon (`quantity * unitPrice`), el `total` de la orden y un `folio` legible con formato `{customerCode}-{id:D5}` (por ejemplo `C00123-00012`).

```http
POST https://bamboonetapi.ddns.net/api/PreOrdenes
```

| Campo | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `customerCode` | string | Si | Codigo de cliente. Max 200 caracteres. |
| `email` | string | No | Email de contacto. Max 200 caracteres, debe ser un email valido. |
| `phone` | string | No | Telefono de contacto. Max 50 caracteres. |
| `notes` | string | No | Notas libres para el vendedor. |
| `items` | array | Si | Se requiere al menos un item. |
| `items[].productCode` | string | Si | Codigo de producto o SKU. Max 50 caracteres. |
| `items[].quantity` | integer | Si | Debe ser mayor a cero. |
| `items[].unitPrice` | decimal | Si | No puede ser negativo. |

Cuerpo de la peticion:

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

Respuesta — `201 Created`. Regresa la pre-orden tal como se guardo, incluyendo el `id` generado, el `folio`, los totales calculados y `createdAt`.

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

### 6.2 Listar pre-ordenes

Lista las pre-ordenes, opcionalmente filtradas por estatus. Los resultados se ordenan por `createdAt` descendente.

```http
GET https://bamboonetapi.ddns.net/api/PreOrdenes?status={status}
```

| Parametro | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `status` | string | No | Filtra por estatus (sin distinguir mayusculas). Ejemplo: `PENDIENTE`. Si se omite, regresa todas. |

### Respuesta

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

### 6.3 Detalle de pre-orden por id

Obtiene el detalle completo de una pre-orden, incluyendo sus items. Regresa `404` si no existe ninguna pre-orden con el `id` dado.

Ademas de los campos base, **cada item incluye el desglose de existencias por almacen de venta** (almacenes con `sales_enabled = 1`), usando el stock entregable (`deliverable_qty`). Esto refleja la vista "Almacen a surtir (stock)" de la pantalla de cotizacion.

```http
GET https://bamboonetapi.ddns.net/api/PreOrdenes/{id}
```

| Parametro | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `id` | integer | Si | Id de la pre-orden. |

Campos de stock por item:

| Campo | Tipo | Descripcion |
| --- | --- | --- |
| `stockDisponible` | integer | Stock entregable total sumando todos los almacenes de venta. |
| `cantidadCubierta` | integer | Cuanto de lo pedido se puede cubrir con el stock disponible. |
| `cantidadAgotada` | integer | Cuanto no se puede cubrir (faltante). |
| `estadoSurtido` | string | `CUBIERTA` (un solo almacen lo cubre todo), `DISTRIBUIR` (hay stock suficiente pero repartido), `AGOTADO_PARCIAL` (falta stock) o `SIN_STOCK` (ningun almacen tiene existencia). |
| `almacenes[]` | array | Solo almacenes con stock, ordenados de mayor a menor. |
| `almacenes[].almacenId` | integer | Id del almacen. |
| `almacenes[].almacen` | string | Nombre del almacen. |
| `almacenes[].stockDisponible` | integer | Piezas entregables en este almacen. |
| `almacenes[].cantidadSurtir` | integer | Piezas sugeridas a surtir desde este almacen (reparto greedy). |

### Respuesta

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

### 6.4 Listar pre-ordenes con detalle (endpoint en ingles)

Lista **todas** las pre-ordenes (opcionalmente filtradas por estatus) con el mismo detalle enriquecido por item que `6.3` (stock por almacen, cantidad cubierta/faltante, estado de surtido y reparto sugerido). El stock de todos los items se resuelve en una sola consulta.

:::info Idioma
Este endpoint esta completamente **en ingles** — ruta, nombres de campos y valores de estado — porque lo revisa el equipo de China. Los demas endpoints de pre-ordenes se mantienen en espanol. Se sirve en una ruta absoluta (`/api/preorders/detail`), fuera del prefijo `/api/PreOrdenes`.
:::

```http
GET https://bamboonetapi.ddns.net/api/preorders/detail?status={status}
```

| Parametro | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `status` | string | No | Filtra por estatus en **ingles**: `PENDING`, `TAKEN`, `CONVERTED`, `CANCELLED`. Si se omite, regresa todas. |

Campos por item (equivalentes en ingles de los campos en espanol de `6.3`):

| Campo | Tipo | Descripcion |
| --- | --- | --- |
| `availableStock` | integer | Stock entregable total en todos los almacenes de venta. |
| `coveredQuantity` | integer | Cuanto de lo pedido se puede cubrir. |
| `shortageQuantity` | integer | Cuanto no se puede cubrir (faltante). |
| `fulfillmentStatus` | string | `COVERED`, `DISTRIBUTE`, `PARTIALLY_COVERED` u `OUT_OF_STOCK`. |
| `warehouses[]` | array | Solo almacenes con stock, ordenados de mayor a menor. |
| `warehouses[].warehouseId` | integer | Id del almacen. |
| `warehouses[].warehouse` | string | Nombre del almacen. |
| `warehouses[].availableStock` | integer | Piezas entregables en este almacen. |
| `warehouses[].quantityToFulfill` | integer | Piezas sugeridas a surtir desde este almacen (reparto greedy). |

El `status` de la pre-orden tambien se regresa en ingles: `PENDING`, `TAKEN`, `CONVERTED`, `CANCELLED`.

### Respuesta

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
