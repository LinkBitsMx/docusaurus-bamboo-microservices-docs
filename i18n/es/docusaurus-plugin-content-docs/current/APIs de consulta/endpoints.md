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

## 7. Ventas (endpoint en ingles)

Consulta detallada de las ventas registradas en BambooERP: cabecera, cliente, totales, facturacion, el detalle renglon por renglon y el estatus de cada almacen.

:::info Idioma
Esta API esta completamente **en ingles** — rutas, nombres de campos y valores de estado — porque la maneja el equipo de China, igual que `6.4`. Los demas endpoints se mantienen en espanol.
:::

### Como se modela una venta

Una venta se guarda en `quotation` (cabecera, totales y **estatus general**) y su detalle en `quotation_detail`, donde cada renglon guarda el almacen que lo surte.

- Mientras la venta esta en **cotizacion**, ese estatus general es el unico que existe: `isQuotation` es `true` y `warehouses[]` regresa vacio.
- Al validarse la cotizacion, **la orden se divide y cada almacen guarda su propio estatus**. En ese momento `isQuotation` pasa a `false` y `warehouses[]` trae el estatus vigente de cada uno.

En la practica, las ventas que aun no se dividen estan en estatus `Sin procesar` y las ya divididas en `Pago Validado`, pero `isQuotation` se calcula por la existencia real de ordenes por almacen, no por el nombre del estatus.

Cada estatus se regresa por duplicado:

| Campo | Descripcion |
| --- | --- |
| `statusRaw` | Nombre del estatus tal cual esta en BambooERP (en espanol). Sirve para rastrear el valor original. |
| `status` | Ese mismo valor normalizado a las etapas en ingles de abajo. |

**Valores posibles de `status`:**

| Valor | Significado |
| --- | --- |
| `IN_QUOTATION` | Aun en cotizacion |
| `SENT_TO_CEDIS` | Pago procesado, se mando a surtir al CEDIS |
| `IN_PICKING` | En proceso de surtido |
| `IN_PACKING_OR_REVIEW` | En proceso de empacado o revision |
| `IN_SHIPPING_LABEL` | En proceso de guias de envio |
| `DELIVERED` | Recolectado o entregado |
| `CANCELLED` | Cancelado |
| `UNKNOWN` | Estatus sin mapear |

### 7.1 Listar ventas

Lista paginada de ventas, ordenadas de la mas reciente a la mas antigua. Cada venta trae su estatus general y, si ya se valido la cotizacion, el estatus de la orden de cada almacen.

```http
GET https://bamboonetapi.ddns.net/api/sales
```

| Parametro | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `startDate` | date | No | Limite inferior de la fecha de venta. Ejemplo: `2026-07-01` |
| `endDate` | date | No | Limite superior de la fecha de venta. Si se manda sin hora, se incluye el dia completo. |
| `customerCode` | string | No | Codigo de cliente exacto. Ejemplo: `SLP2A101255` |
| `folio` | string | No | Coincidencia parcial del folio. Ejemplo: `2607-` |
| `statusId` | integer | No | Id del estatus general. Ejemplo: `27` (Pago Validado) |
| `warehouseId` | integer | No | Solo ventas con renglones surtidos por ese almacen. |
| `onlyQuotations` | boolean | No | `true` deja unicamente las ventas que siguen en cotizacion. |
| `page` | integer | No | Numero de pagina. Default `1`. |
| `pageSize` | integer | No | Tamano de pagina. Default `50`, maximo `200`. |

Ejemplo con filtros:

```http
GET https://bamboonetapi.ddns.net/api/sales?startDate=2026-07-01&endDate=2026-07-31&pageSize=50
```

### Respuesta

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

### 7.2 Detalle de venta

Obtiene el detalle completo de una venta por folio. Regresa `404` si no existe ninguna venta con ese folio.

Ademas de la cabecera y el detalle completo, `warehouses[]` resume cada orden por almacen (estatus, piezas, renglones e importe que le tocan) y cada item indica el almacen que lo surte junto con el estatus de esa orden.

```http
GET https://bamboonetapi.ddns.net/api/sales/{folio}
```

| Parametro | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `folio` | string | Si | Folio de la venta. Ejemplo: `2607-00037` |

Totales:

| Campo | Tipo | Descripcion |
| --- | --- | --- |
| `units` | integer | Piezas sumadas de los renglones de producto. |
| `totalLines` | integer | Numero de renglones del detalle (productos y servicios). |
| `productsSubtotal` | decimal | Suma de los renglones de producto. |
| `servicesTotal` | decimal | Suma de los renglones de servicio (envio, flete, etc.). |
| `lineDiscount` | decimal | Descuento sumado de los renglones. |
| `deliveryTotal` | decimal | Columna `total_deliver` de `quotation`. |
| `assuredTotal` | decimal | Columna `total_of_assured` de `quotation`. |
| `freightCarrierTotal` | decimal | Columna `total_fletera` de `quotation`. |
| `total` | decimal | Total con el que cierra la venta (columna `total`). |
| `initialTotal` | decimal | Total antes de modificaciones (columna `total_initial`). |
| `hasDiscount` | boolean | Si la venta trae descuento. |

Campos por almacen:

| Campo | Tipo | Descripcion |
| --- | --- | --- |
| `warehouses[].warehouseId` | integer | Id del almacen. |
| `warehouses[].warehouse` | string | Nombre del almacen. |
| `warehouses[].statusRaw` | string | Estatus de esa orden tal cual esta en BambooERP. |
| `warehouses[].status` | string | Ese mismo estatus normalizado en ingles. |
| `warehouses[].assignedAt` | datetime | Cuando se asigno la orden al almacen. |
| `warehouses[].units` | integer | Piezas que surte este almacen. |
| `warehouses[].totalLines` | integer | Renglones que surte este almacen. |
| `warehouses[].amount` | decimal | Importe de esos renglones. |

Campos por item:

| Campo | Tipo | Descripcion |
| --- | --- | --- |
| `items[].productCode` | string | Codigo interno del producto. |
| `items[].sku` | string | SKU del producto. |
| `items[].product` | string | Nombre del producto. |
| `items[].quantity` | integer | Cantidad pedida. |
| `items[].unitPrice` | decimal | Precio unitario. |
| `items[].discount` | decimal | Descuento aplicado al renglon. |
| `items[].amount` | decimal | Importe del renglon (`quantity * unitPrice`). |
| `items[].isService` | boolean | `true` en renglones de servicio como envio o flete. |
| `items[].warehouseId` | integer | Almacen que surte el renglon (`null` en servicios). |
| `items[].warehouse` | string | Nombre del almacen. |
| `items[].warehouseStatus` | string | Estatus de esa orden por almacen (`null` mientras sigue en cotizacion). |
| `items[].notes` | string | Notas del renglon. |

### Respuesta

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
  ]
}
```
