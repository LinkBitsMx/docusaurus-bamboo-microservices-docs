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
  "productoId": "1871813",
  "codigo": "003553",
  "nombre": "BOCINA X1",
  "sku": "X1",
  "descripcion": "*	Bocina mini portátil
*	Conectividad: Bluetooth con función TWS
*	Potencia de salida: 3W
*	Batería: Litio 300 mAh, 3.7V
*	Colores disponibles: Negro, rosa, morado",
  "preciosPorSucursal": [
    {
      "sucursal": "México",
      "precioMayoreo": 21.5,
      "precioCaja": 18.0,
      "moneda": "MXN",
      "incluyeIva": true
    },
    {
      "sucursal": "Sucursal Monterrey",
      "precioMayoreo": 22.0,
      "precioCaja": 18.5,
      "moneda": "MXN",
      "incluyeIva": true
    }
  ]
}
```

| Campo | Tipo | Descripcion |
| --- | --- | --- |
| `productoId` | string | Id interno del producto. |
| `codigo` | string | Codigo interno. |
| `nombre` | string | Nombre del producto. |
| `sku` | string | SKU (modelo). |
| `descripcion` | string? | Descripcion del producto (`starnet_products.descrip_corta`): la lista de caracteristicas de la ficha del producto, en un solo texto. Conserva los saltos de linea (`
`) y las viñetas `*` tal como se capturaron en el ERP. `null` cuando el producto no tiene descripcion. |
| `preciosPorSucursal` | array | Precio de mayoreo (`precioMayoreo`) y de caja (`precioCaja`) en las sucursales México y Monterrey. |

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

### Sucursal y departamento

La venta llega a su sucursal a traves de su departamento:

```text
quotation.DepartamentoId -> departments.id
departments.branchId     -> starnet_branches.id
```

Filtra con `branchCode` en `7.1`, usando `starnet_branches.code` (por ejemplo `801.10.02` para Sucursal Florida). Una sucursal agrupa varios departamentos, asi que `801.01.01` (REGIONES) cubre juntas las ventas de Oficina, Rutas, Region Noreste y AEK.

El listado regresa `branchCode` y `branch` (nombre). El detalle regresa un objeto `branch` y el `department` del que proviene:

| Campo | Tipo | Descripcion |
| --- | --- | --- |
| `branch.branchId` | integer | Id de la sucursal (`starnet_branches.id`). |
| `branch.code` | string | Codigo de la sucursal — el valor que toma el filtro `branchCode`. Ejemplo: `801.10.02` |
| `branch.name` | string | Nombre de la sucursal. Ejemplo: `Sucursal Florida` |
| `department.departmentId` | integer | Id del departamento (`departments.id`). |
| `department.code` | string | Codigo del departamento. Ejemplo: `801010302` |
| `department.name` | string | Nombre del departamento. |
| `department.zone` | string | Zona a la que pertenece. Ejemplo: `CDMX`, `GDL`, `MTY` |

:::note
Algunas ventas no resuelven sucursal: o no tienen departamento, o apuntan a un id de departamento que ya no existe en `departments`. En esos casos `branch` y `department` regresan `null`.
:::

### Vendedor y pagos

El vendedor que registro la venta sale de `quotation.usuarioId` con el join contra `catUsers`. El listado regresa `sellerId` y `seller` (nombre completo); el detalle regresa un objeto `seller` con codigo, correo y usuario.

El detalle tambien regresa `payments[]`, los pagos registrados contra la venta con su forma de pago. La relacion es:

```text
quotation.id                     -> rel_quotes_to_payments.Quote_id
rel_quotes_to_payments.VoucherId -> Payments.Id
```

:::note
El pago viaja en `VoucherId`, no en `payment_id`. Las filas con `VoucherId = 0` son placeholders sin pago y se omiten.
:::

| Campo | Tipo | Descripcion |
| --- | --- | --- |
| `payments[].paymentId` | integer | Id del pago (`Payments.Id`). |
| `payments[].folio` | string | Folio del pago. Ejemplo: `PAY-0726-000010` |
| `payments[].paymentDate` | datetime | Fecha del pago. |
| `payments[].amount` | decimal | Monto de este pago. |
| `payments[].paymentFormCode` | string | Codigo SAT de la forma de pago (`sat_FormaPago.vchCode`). Ejemplo: `03` |
| `payments[].paymentForm` | string | Nombre de la forma de pago. Ejemplo: `Transferencia electronica de fondos` |
| `payments[].statusRaw` | string | Estatus del pago tal cual esta en BambooERP (en espanol). |
| `payments[].status` | string | `VALID`, `REJECTED`, `PENDING`, `IN_PROCESS`, `CANCELLED` o `UNKNOWN`. |
| `payments[].reference` | string | Referencia bancaria, cuando se registro. |
| `payments[].paymentType` | string | Tipo de pago tal cual se guarda. Ejemplo: `payment` |
| `payments[].createdAt` | datetime | Cuando se ligo el pago a la venta. |

Una venta puede traer varios pagos con formas distintas — por ejemplo una transferencia, varios saldos a favor y efectivo en la misma venta. `totals.paymentsTotal` suma `payments[].amount`.

:::warning
`paymentsTotal` **no** tiene por que coincidir con `total`: una venta puede quedar parcialmente pagada o traer pagos registrados por encima de su total. No lo uses para dar por liquidada una venta.
:::

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
| `statusId` | integer | No | Estatus **general** de la venta (`quotation.status_id`). Solo toma 5 valores — ver el aviso de abajo. |
| `warehouseStatusId` | integer | No | Estatus de las **ordenes por almacen**: deja las ventas donde al menos un almacen esta hoy en ese estatus. Ejemplo: `21` (Recolectado) |
| `warehouseId` | integer | No | Solo ventas con renglones surtidos por ese almacen. |
| `branchCode` | string | No | Codigo de sucursal (`starnet_branches.code`). Ejemplo: `801.10.02` (Sucursal Florida) |
| `onlyQuotations` | boolean | No | `true` deja unicamente las ventas que siguen en cotizacion. |
| `includePayments` | boolean | No | `true` agrega `payments[]` y `paymentsTotal` a cada venta del listado. Default `false`. |
| `page` | integer | No | Numero de pagina. Default `1`. |
| `pageSize` | integer | No | Tamano de pagina. Default `50`, maximo `200`. |

:::warning `statusId` no es la etapa de surtido
`quotation.status_id` solo guarda 5 valores: `1` Sin procesar, `27` Pago Validado, `23` Cancelado, `28` Pago no valido y `29` Pago sin proceso.

Las etapas de surtido — `21` Recolectado, `18` Guia Generada, `15` Empacado Finalizado y las demas — viven en las **ordenes por almacen**, asi que `statusId=21` siempre regresa 0 ventas. Usa `warehouseStatusId=21` en su lugar.
:::

Por default el listado **no** incluye los pagos, porque cuestan una consulta extra por pagina. Pidelos con `includePayments=true` y cada venta gana `payments[]` (los mismos campos del detalle) mas `paymentsTotal`. El endpoint de detalle siempre los regresa.

Ejemplo con filtros:

```http
GET https://bamboonetapi.ddns.net/api/sales?startDate=2026-07-01&endDate=2026-07-31&pageSize=50
GET https://bamboonetapi.ddns.net/api/sales?branchCode=801.01.01&startDate=2026-05-01&endDate=2026-06-01&warehouseStatusId=21&includePayments=true
```

:::note
Todos los filtros viajan en el **query string**. Este es un `GET`: los filtros enviados como cuerpo JSON se ignoran y la peticion regresa como si no llevara filtros.
:::

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
| `paymentsTotal` | decimal | Suma de `payments[].amount` — lo realmente pagado. |
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

## 8. Pagos (endpoint en ingles)

Registra pagos en BambooERP (`Payments`), la misma tabla que escribe el ERP cuando se sube un comprobante. Tambien expone un endpoint de lectura para recuperar el pago por id.

:::info Idioma
Esta API esta completamente **en ingles** — rutas, nombres de campos y valores de estado — porque la maneja el equipo de China, igual que `6.4` y `7`. Los demas endpoints se mantienen en espanol.
:::

### Como se registra un pago

El pago se crea **pendiente de validacion**: estatus `4` (PENDIENTE) y todavia sin monto. El monto y el estatus final (`29` Valido / `30` Rechazado) los asigna despues quien lo valida dentro del ERP. Aun asi puedes mandar `amount` desde el alta, y `statusId` si necesitas cambiar el valor por defecto.

Hay dos cosas que hace la base de datos por si sola, asi que **no** van en la peticion:

| Lo hace | Que hace |
| --- | --- |
| Trigger `trg_GenerarFolioPayments` | Genera el `folio` con formato `PAY-MMYY-NNNNNN`, consecutivo por mes. Por eso el `folio` no se manda en el body. |
| Trigger `trg_AfterInsert_Payments_InsertRelation` | Relaciona el pago con la venta en `rel_quotes_to_payments` cuando se manda `saleFolio`. |

:::warning Mandar `saleFolio` mueve la venta
Cuando el pago se relaciona con una venta, la base de datos tambien mueve esa venta a estatus `29`. Omite `saleFolio` si solo quieres registrar el pago sin tocar la venta.
:::

La empresa receptora tampoco se manda: se deriva de la cuenta bancaria.

```text
bankId -> bancos.id
bancos.id_origen -> origen_cuenta.id   (accountId en la respuesta)
```

Cada estatus se regresa por duplicado, igual que en `7`: `statusRaw` es el valor tal cual esta en BambooERP (en espanol) y `status` es ese mismo valor normalizado a `VALID`, `REJECTED`, `PENDING`, `IN_PROCESS`, `CANCELLED` o `UNKNOWN`.

### Campos de Kingdee

El pago puede traer los campos del documento de recarga de Kingdee (充值单). Seis de ellos ya salen de lo que el pago guarda hoy y **no** se mandan: se derivan de los catalogos de BambooERP. El resto son identificadores propios de Kingdee, y como Bamboo no tiene catalogo contra el cual resolverlos, se guardan en `Payments` tal cual llegan.

| Campo Kingdee | Campo del body | De donde sale |
| --- | --- | --- |
| `FBillNo` (单据编号) | `kingdeeBillNo` | Se manda. No sustituye a `folio`, que lo sigue generando la base de datos. |
| `FDate` (单据日期) | — | `paymentDate` |
| `FBizOrgId` / `FBizOrg` (业务组织) | `bizOrgId` / `bizOrgCode` | Se manda |
| `FSETTLEORGID` / `FSETTLEORG` (结算组织) | `settleOrgId` / `settleOrgCode` | Se manda |
| `FBranchID` / `Fbranch` (充值门店) | — | `departmentId` → `departments.branchId` → `starnet_branches.id` / `.code` |
| `FSalerID` / `FSaler` (业务员) | — | `sellerId` → `catUsers.code_seller`, guardado como `kingdeeId_kingdeeCode_branchId` |
| `FCashierID` / `FCashier` (收银员) | `cashierId` / `cashierCode` | Se manda. Es el cajero de Kingdee, que no necesariamente es `uploadedById`. |
| `FCustomerID` / `FCustomer` (客户) | — | `customerCode` → `customers.customer_id` / `customer_code` |
| `FSETTLECURRENCYID` / `FSETTLECURRENCY` (结算币别) | `settleCurrencyId` / `settleCurrencyCode` | Se manda |
| `FNote` (备注) | — | `comentary` |
| `FCardID` / `FCard` (卡号) | `cardId` / `cardNumber` | Se manda |
| `FMemberID` / `FMember` (会员卡号) | `memberId` / `memberCardNumber` | Se manda |
| `FAccountID` / `FAccount` (账户) | `kingdeeAccountId` / `kingdeeAccountCode` | Se manda. Es la cuenta de Kingdee, no el `accountId` de la respuesta, que es la empresa receptora. |
| `FRechargeAmount` (充值金额) | `rechargeAmount` | Se manda. Es lo que se abona a la tarjeta, a diferencia de `amount`, que es lo que se cobro. |
| `FReceiveTypeID` / `FReceiveType` (收款方式) | `receiveTypeId` / `receiveTypeCode` | Se manda. Es la forma de cobro de Kingdee, independiente de `paymentFormId` (la forma de pago del SAT). |
| `FReceiveCurrencyID` / `FReceiveCurrency` (收款币别) | `receiveCurrencyId` / `receiveCurrencyCode` | Se manda. Por defecto toma la moneda de liquidacion. |
| `FReceiveAmt` (收款金额) | — | `amount` |
| `FExchangeRate` (汇率) | `exchangeRate` | Se manda. Por defecto `1` mientras ambas monedas coincidan; **obligatorio cuando difieren**, porque BambooERP no tiene tabla de tipo de cambio. |

Todos son opcionales, asi que el body que ya manda el ERP sigue funcionando sin cambios. La respuesta incluye el bloque `kingdee` con el documento armado y los nombres `F*` escritos exactamente como los espera Kingdee (es la unica parte de la API que no va en `camelCase`).

### 8.1 Registrar pago

```http
POST https://bamboonetapi.ddns.net/api/payments
```

| Campo | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `customerCode` | string | Si | Cliente al que pertenece el pago (`customers.customer_code`). Max 50 caracteres. |
| `bankId` | integer | Si | Cuenta bancaria donde se deposito (`bancos.id`). Debe existir y no estar deshabilitada. |
| `paymentFormId` | integer | Si | Forma de pago (`sat_FormaPago.ID`). Ejemplo: `3` = transferencia. |
| `uploadedById` | integer | Si | Usuario que registra el pago (`catUsers.idUsuario`). |
| `paymentDate` | date | No | Fecha del pago. Por defecto la fecha de hoy. |
| `amount` | decimal | No | Monto. Debe ser mayor a cero cuando se manda. Se deja vacio hasta la validacion, igual que en el ERP. |
| `reference` | string | No | Referencia bancaria de la transferencia o deposito. Max 250 caracteres. |
| `paymentType` | string | No | `payment` (default), `credit` o `advance`. |
| `paymentFilePath` | string | No | Ruta del archivo del comprobante. Max 250 caracteres. |
| `saleFolio` | string | No | Folio de la venta a la que se aplica el pago (`quotation.billCode`). Ver la advertencia de arriba. |
| `sellerId` | integer | No | Vendedor al que se acredita el pago (`catUsers.idUsuario`). |
| `departmentId` | integer | No | Sucursal a la que pertenece el pago (`departments.id`). Por defecto la sucursal de `uploadedById`. |
| `statusId` | integer | No | Estatus con el que se crea el pago (`catEstatus.idEstatus`). Por defecto `4` (PENDIENTE). |
| `comentary` | string | No | Comentario. Max 1500 caracteres. |
| `observations` | string | No | Observaciones. Max 500 caracteres. |

Mas los campos de Kingdee descritos arriba, todos opcionales: `kingdeeBillNo`, `bizOrgId`, `bizOrgCode`, `settleOrgId`, `settleOrgCode`, `cashierId`, `cashierCode`, `kingdeeAccountId`, `kingdeeAccountCode`, `receiveTypeId`, `receiveTypeCode`, `settleCurrencyId`, `settleCurrencyCode`, `receiveCurrencyId`, `receiveCurrencyCode`, `exchangeRate`, `cardId`, `cardNumber`, `memberId`, `memberCardNumber` y `rechargeAmount`.

Body de la peticion:

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

Respuesta — `201 Created`. Regresa el pago tal cual quedo guardado, incluyendo el `folio` que genero la base de datos.

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
  },
  "kingdeeSales": [
    {
      "saleId": 201363,
      "isPos": false,
      "folio": "XSCKD100949",
      "amountApplied": 5166.67,
      "appliedDate": "2026-05-18T17:29:44.433"
    }
  ]
}
```

#### `kingdeeSales`: el folio de la venta en Kingdee

Las ventas contra las que se aplico el pago salen de `PaymentApplications`, y el folio del documento que se genero en Kingdee vive en una de dos tablas, segun `isPOS`:

| `isPOS` | Tabla | Se busca por | Folio |
| --- | --- | --- | --- |
| `0` | `kingdee_sales_invoices` | `id = SaleId` | `bill_code`. Ejemplo: `XSCKD100949` |
| `1` | `KingDeeSalesPOS` | `Id = SaleId` | `Folio`. Ejemplo: `10040002604109633` |

Es una **lista**, no un campo unico: un mismo pago se puede repartir entre varias ventas. El pago `PAY-0526-000011`, por ejemplo, esta aplicado a tres facturas (`XSCKD100949`, `XSCKD100955`, `XSCKD100950`), cada una con su propio `amountApplied`.

:::note `saleId: 0` significa que la venta todavia no existe en Kingdee
El pago ya esta aplicado, pero la venta no se ha generado del lado de Kingdee, asi que `folio` regresa en `null`. La aplicacion se publica de todas formas en lugar de omitirse, para distinguirla de un pago que nunca se aplico — ese regresa `kingdeeSales` vacio.
:::

Todas las referencias se validan contra BambooERP antes de insertar el pago. Si alguna no existe, no se escribe nada y la API regresa `400` con el motivo:

```json
{ "message": "Customer 'NO-EXISTE' not found." }
```

| Caso | Mensaje |
| --- | --- |
| Cliente no encontrado | `Customer '{customerCode}' not found.` |
| Cuenta bancaria no encontrada | `Bank account {bankId} not found.` |
| Cuenta bancaria deshabilitada | `Bank account {bankId} is disabled.` |
| Forma de pago no encontrada | `Payment form {paymentFormId} not found.` |
| Usuario, vendedor, departamento o estatus no encontrado | `{Entidad} {id} not found.` |
| Folio de venta no encontrado | `Sale '{saleFolio}' not found.` |
| Tipo de pago invalido | `paymentType must be one of: payment, credit, advance.` |
| Monedas distintas sin tipo de cambio | `exchangeRate is required when settleCurrencyCode and receiveCurrencyCode differ.` |

### 8.2 Consultar pago

Obtiene un pago por id, con la misma estructura de la respuesta anterior. Regresa `404` si no existe un pago con ese id.

```http
GET https://bamboonetapi.ddns.net/api/payments/{id}
```

| Parametro | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `id` | integer | Si | Id del pago (`Payments.Id`). Ejemplo: `34076` |

### 8.3 Listar pagos por estatus

Listado paginado de pagos, del mas nuevo al mas viejo. El estatus se **filtra y se publica en ingles**, nunca con el nombre en espanol que BambooERP guarda en `catEstatus`.

```http
GET https://bamboonetapi.ddns.net/api/payments
```

| `status` | `catEstatus` | Que es |
| --- | --- | --- |
| `VALID` | `29` Valido | Pago validado |
| `REJECTED` | `30` Rechazado | Pago rechazado |
| `PENDING` | `4` PENDIENTE | Registrado, todavia sin revisar — es el estatus con el que nace un pago |
| `IN_PROCESS` | `17` EN PROCESO | En proceso de validacion |
| `CANCELLED` | `8` CANCELADO | Cancelado |

Se pueden pedir varios a la vez separados por coma. Si se omite `status`, entran todos.

```http
GET .../api/payments?status=REJECTED
GET .../api/payments?status=REJECTED,IN_PROCESS
GET .../api/payments?status=VALID&startDate=2026-08-01&endDate=2026-08-07
GET .../api/payments?status=VALID&customerCode=SLP2A101255&pageSize=100
GET .../api/payments?branchCode=801.10.02&status=VALID
GET .../api/payments?saleFolio=2608-00022
GET .../api/payments?kingdeeSaleFolio=XSCKD100949
GET .../api/payments?saleStartDate=2026-07-01&saleEndDate=2026-07-31
GET .../api/payments?kingdeeSaleStartDate=2026-05-01&kingdeeSaleEndDate=2026-05-31
```

:::note Dos folios de venta, dos filtros
`saleFolio` es la venta de BambooERP — la que se publica en la respuesta como `saleFolio`. `kingdeeSaleFolio` es el documento generado en Kingdee — el que aparece en `kingdeeSales[].folio`. Son identificadores distintos de la misma operacion comercial, por eso cada uno tiene su propio filtro.

Combinar `kingdeeSaleFolio` con `kingdeeSaleStartDate`/`kingdeeSaleEndDate` exige que **la misma venta** cumpla ambas condiciones: un pago repartido entre varias facturas no entra por tener el folio en una y la fecha en otra.
:::

:::note Tres rangos de fecha, tres fechas distintas
`startDate`/`endDate` filtran por **cuando se hizo el pago**, `saleStartDate`/`saleEndDate` por **cuando se registro la venta en BambooERP**, y `kingdeeSaleStartDate`/`kingdeeSaleEndDate` por **cuando se facturo la venta en Kingdee**. Son independientes y se combinan con AND, asi que se puede pedir los pagos cobrados en agosto de ventas facturadas en Kingdee en mayo.

El rango `kingdeeSale*` consulta `PaymentApplications`, `kingdee_sales_invoices` y `KingDeeSalesPOS`. En un ambiente sin esas tablas ese filtro falla; el resto del endpoint no las toca.
:::

:::note Un estatus mal escrito es un error, no una pagina vacia
Un valor fuera de la tabla regresa `400` con la lista de los validos, para que un typo no se lea como "no hay ninguno":

```json
{ "message": "Unknown status: RECHAZADO. status must be one of: VALID, REJECTED, PENDING, IN_PROCESS, CANCELLED." }
```
:::

| Parametro | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `status` | string | No | Uno o varios estatus en ingles, separados por coma. Ejemplo: `REJECTED,IN_PROCESS` |
| `statusId` | integer | No | Estatus por id interno (`catEstatus.idEstatus`), para quien trabaje con el id. Se combina con `status`. |
| `customerCode` | string | No | Codigo exacto del cliente (`customers.customer_code`). Ejemplo: `SLP2A101255` |
| `folio` | string | No | Coincidencia parcial sobre el folio del pago. Ejemplo: `PAY-0826` |
| `saleFolio` | string | No | Folio de la venta de **BambooERP** a la que se aplico el pago (`quotation.billCode`). Coincidencia exacta. Ejemplo: `2608-00022` |
| `kingdeeSaleFolio` | string | No | Folio de la venta tal como se genero en **Kingdee**: `kingdee_sales_invoices.bill_code` (ejemplo: `XSCKD100949`) o `KingDeeSalesPOS.Folio` si es ticket POS (ejemplo: `10040002604109633`). Coincidencia exacta. |
| `branchCode` | string | No | Sucursal **del pago** (`starnet_branches.code`), via `Payments.DepartmentId` → `departments.branchId`. Es la misma que se publica en `kingdee.Fbranch`. Ejemplo: `801.10.02` |
| `startDate` | date | No | Limite inferior de la **fecha de pago** (`paymentDate`). Ejemplo: `2026-08-01` |
| `endDate` | date | No | Limite superior de la **fecha de pago**. Si se manda sin hora, se incluye el dia completo. |
| `saleStartDate` | date | No | Limite inferior de la **fecha de la venta de BambooERP** (`quotation.created_at`, la venta detras de `saleFolio`). Deja fuera los pagos sin venta relacionada. |
| `saleEndDate` | date | No | Limite superior de la fecha de la venta de BambooERP. Dia completo incluido. |
| `kingdeeSaleStartDate` | date | No | Limite inferior de la **fecha de la venta en Kingdee**: `kingdee_sales_invoices.bill_date`, o `KingDeeSalesPOS.BillDate` cuando la venta es un ticket POS. Conserva el pago si al menos una de sus aplicaciones cae en el rango. |
| `kingdeeSaleEndDate` | date | No | Limite superior de la fecha de la venta en Kingdee. Dia completo incluido. |
| `paymentFormId` | integer | No | Forma de pago SAT (`sat_FormaPago.ID`). |
| `bankId` | integer | No | Cuenta bancaria (`bancos.id`). |
| `departmentId` | integer | No | Sucursal o departamento del pago (`departments.id`). |
| `sellerId` | integer | No | Vendedor al que se acredita el pago (`catUsers.idUsuario`). |
| `paymentType` | string | No | `payment`, `credit` o `advance`. |
| `page` | integer | No | Numero de pagina. Default `1`. |
| `pageSize` | integer | No | Tamano de pagina. Default `50`, maximo `200`. |

:::note
Todos los filtros viajan en el **query string**. Esto es un `GET`: los filtros enviados como cuerpo JSON se ignoran y la peticion regresa como si no tuviera filtros.
:::

### Respuesta

Cada elemento de `payments[]` es el **registro completo del pago**, identico al que regresa `8.2` — incluido el bloque `kingdee`. Ademas, `summary` desglosa **todo lo que matcheo el filtro**, no solo la pagina actual, y `totalAmount` es la suma de esos importes.

```json
{
  "page": 1,
  "pageSize": 50,
  "totalRecords": 1002,
  "totalPages": 21,
  "totalAmount": 6132011.20,
  "summary": [
    {
      "statusId": 17,
      "statusRaw": "EN PROCESO",
      "status": "IN_PROCESS",
      "count": 5,
      "amount": 0.00
    },
    {
      "statusId": 30,
      "statusRaw": "Rechazado",
      "status": "REJECTED",
      "count": 997,
      "amount": 6132011.20
    }
  ],
  "payments": [
    {
      "paymentId": 34091,
      "folio": "PAY-0826-000039",
      "customerCode": "SLP2A101255",
      "customer": "JESUS ADIEL DOMINGO MONSIVAIS",
      "paymentDate": "2026-08-07T00:00:00",
      "amount": 6.00,
      "paymentFormId": 1,
      "paymentFormCode": "01",
      "paymentForm": "Efectivo",
      "statusId": 29,
      "statusRaw": "Valido",
      "status": "VALID",
      "saleId": 79032,
      "saleFolio": "2608-00022",
      "kingdee": { "FBillNo": null, "FDate": "2026-08-07T00:00:00" }
    }
  ]
}
```

:::warning `amount` viene vacio hasta que el pago se valida
El ERP llena el importe cuando alguien valida el pago, por eso `PENDING` e `IN_PROCESS` suman `0.00` en `summary` aunque si tengan pagos. Ese es el dato real, no un error de la suma: solo `VALID` y `REJECTED` traen importes.
:::

`summary` lista unicamente los estatus presentes en el conjunto filtrado: pedir `status=REJECTED` regresa una sola entrada, y un filtro que no matchea nada regresa `totalRecords: 0` con `summary` y `payments` vacios.

## 9. Ventas a credito (endpoint en ingles)

Las ventas que se fueron a credito, cada una con los pagos aplicados en su contra, mas las metricas de todo lo que matcheo el filtro: cuantas ventas a credito hay y cuanto tarda el cliente en pagarlas.

:::info Idioma
Esta API esta completamente **en ingles** — rutas, nombres de campos y valores de estado — igual que `7` y `8`.
:::

### Como se modela una venta a credito

Una venta a credito **no es la `quotation`**. Es el documento generado en Kingdee — `kingdee_sales_invoices` con `isCredit = 1` y sin cancelar — que es la misma definicion que usa el propio ERP en su vista `ordersCredits`, y el unico lugar donde viven el saldo y la fecha de liquidacion de la venta.

```text
kingdee_sales_invoices.quoteId -> quotation.id
```

La venta de BambooERP — la que publica `7` — se alcanza por `quoteId`, asi que cada registro trae **dos folios de la misma operacion comercial**:

| Campo | Descripcion |
| --- | --- |
| `folio` | Folio de la venta en Kingdee (`kingdee_sales_invoices.bill_code`). Ejemplo: `XSCKD166673` |
| `saleFolio` | Folio de la misma venta en BambooERP (`quotation.billCode`). Ejemplo: `2605-05331` |
| `invoiceCode` | Folio de la factura fiscal, cuando la venta se facturo. Ejemplo: `MEX79556` |

El estado de la deuda sale directo del documento de Kingdee:

| Campo | Descripcion |
| --- | --- |
| `total` | Importe por el que se facturo la venta (`bill_total_amount`). |
| `paid` | Ya cobrado: `total - balance`. |
| `balance` | Lo que aun se debe. |
| `settledAt` | Fecha en que la venta quedo saldada (`conclusion_date`). Es `null` mientras se deba. |

### Plazo y fecha de vencimiento

Los dias de credito salen de la linea de credito del cliente (`CreditLines.creditDays`) y del proceso de su solicitud (`credits.processType`). `dueDate` y `daysRemaining` se calculan **exactamente como lo hace el ERP**, para que la API y la pantalla de creditos nunca discrepen:

| Proceso del cliente | Plazo |
| --- | --- |
| `Proceso CheckPlus` | `creditDays` planos desde `billDate` |
| Todos los demas | `creditDays + 3`, **sin contar domingos** |

### Estatus

El estatus se calcula, no se guarda, y se publica en ingles:

| `status` | ERP (`ordersCredits`) | Significado |
| --- | --- | --- |
| `PAID` | `Pagada` | `balance = 0`: la venta ya se liquido |
| `OVERDUE` | `Pago vencido` | Aun se debe y ya paso su fecha de vencimiento |
| `PENDING` | `Pendiente de pago` | Aun se debe, pero dentro del plazo |

Cada estatus se regresa por duplicado, igual que en `7`: `statusRaw` es el nombre que calcula el ERP (en espanol) y `status` es ese mismo valor en ingles.

:::note Una venta liquidada puede seguir marcando vencimiento
`status: "PAID"` con `daysOverdue > 0` no es una contradiccion: la venta se pago, pero despues de su fecha de vencimiento. Es la semantica del propio ERP y se conserva tal cual.
:::

### Los abonos

Los pagos son las aplicaciones de un pago contra el documento de Kingdee:

```text
PaymentApplications.SaleId    -> kingdee_sales_invoices.id   (StatusId = 1, isPOS = 0)
PaymentApplications.PaymentId -> Payments.Id
```

**Un pago se puede repartir entre varias ventas**, asi que lo que se publica por venta es la parte aplicada a ella, no el pago completo:

| Campo | Tipo | Descripcion |
| --- | --- | --- |
| `payments[].paymentId` | integer | Id del pago (`Payments.Id`). |
| `payments[].folio` | string | Folio del pago. Ejemplo: `PAY-0626-000900` |
| `payments[].amount` | decimal | Importe del pago **aplicado a esta venta** (`AmountApplied`). |
| `payments[].paymentAmount` | decimal | Importe del pago completo, que puede cubrir otras ventas tambien. |
| `payments[].paymentDate` | datetime | Cuando pago el cliente. |
| `payments[].appliedDate` | datetime | Cuando el ERP aplico el pago a esta venta. |
| `payments[].daysFromSale` | integer | Dias de `billDate` a `paymentDate`. Negativo cuando el cliente pago por anticipado. |
| `payments[].paymentFormCode` | string | Codigo SAT de la forma de pago (`sat_FormaPago.vchCode`). Ejemplo: `03` |
| `payments[].paymentForm` | string | Nombre de la forma de pago. Ejemplo: `Transferencia electronica de fondos` |
| `payments[].bank` | string | Banco del pago, cuando esta registrado. |
| `payments[].reference` | string | Referencia bancaria, cuando esta registrada. |
| `payments[].paymentType` | string | `payment`, `credit` o `advance`. |
| `payments[].statusRaw` | string | Estatus del pago tal cual esta en BambooERP (en espanol). |
| `payments[].status` | string | `VALID`, `REJECTED`, `PENDING`, `IN_PROCESS`, `CANCELLED` o `UNKNOWN`. |

:::note Las tablas legacy no se leen
`applyPaymentsCredits` y `paymentsCredits` son el par viejo de pagos a credito. Sus renglones se migraron a `PaymentApplications`, que es donde el ERP escribe hoy, asi que el endpoint solo lee esta ultima. Las aplicaciones que el ERP deshizo (`StatusId <> 1`) quedan fuera.
:::

### Cuanto tarda un cliente en pagarnos

Hay **dos maneras de medirlo y el endpoint publica las dos**, porque no dan la misma respuesta:

| Metrica | Se mide sobre | Que responde |
| --- | --- | --- |
| `daysToSettle` | `billDate` → `settledAt` | El dato del ERP: cuando se marco la venta como saldada. Es lo que muestra la pantalla de creditos. |
| `daysToLastPayment` | `billDate` → `paymentDate` del ultimo abono | Cuando pago **el cliente** de verdad. |

La diferencia entre ambas es el rezago del propio ERP en aplicar los pagos, no del cliente. Por eso mismo cada abono trae `paymentDate` y `appliedDate` como campos separados.

:::warning Como leer los promedios
- **`daysToSettle` de una venta sin liquidar cuenta hasta hoy**, asi que se lee como "cuanto lleva abierta". Para promediar solo lo que ya se cobro esta `avgDaysToSettle`, que toma unicamente las liquidadas; para lo que sigue abierto, `avgDaysOutstanding`.
- **`weightedAvgDaysToSettle` pondera cada venta por su importe**, asi que una factura grande pagada tarde pesa mas que una chica. Ese es el numero a leer como DSO — el promedio simple trata igual una venta de $500 que una de $500,000.
:::

### 9.1 Listar ventas a credito

Listado paginado de ventas a credito, de la mas nueva a la mas vieja.

```http
GET http://pfconexionlinkbits.ddns.net:50780/api/credit-sales
```

```http
GET .../api/credit-sales?status=OVERDUE
GET .../api/credit-sales?status=OVERDUE,PENDING&includePayments=false
GET .../api/credit-sales?customerCode=AAA2A102159
GET .../api/credit-sales?startDate=2026-05-01&endDate=2026-05-31&status=PAID
GET .../api/credit-sales?minDaysToSettle=60
GET .../api/credit-sales?folio=XSCKD166
GET .../api/credit-sales?saleFolio=2605-05331
```

| Parametro | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `status` | string | No | Uno o varios estatus en ingles, separados por coma. Ejemplo: `OVERDUE,PENDING` para todo lo que sigue debiendose. |
| `startDate` | date | No | Limite inferior de la **fecha de la venta** (`billDate`). Ejemplo: `2026-05-01` |
| `endDate` | date | No | Limite superior de la fecha de la venta. Si se manda sin hora, se incluye el dia completo. |
| `customerCode` | string | No | Codigo exacto del cliente (`customers.customer_code`). Ejemplo: `AAA2A102159` |
| `folio` | string | No | Coincidencia parcial sobre el folio de Kingdee. Ejemplo: `XSCKD166` |
| `saleFolio` | string | No | Folio de la venta en BambooERP (`quotation.billCode`). Coincidencia exacta. Ejemplo: `2605-05331` |
| `branchCode` | string | No | Sucursal de la venta en Kingdee (`starnet_branches.code` via `kingdee_sales_invoices.branch_id`). Ejemplo: `801` |
| `sellerId` | integer | No | Vendedor de la venta (`quotation.usuarioId`). |
| `minDaysToSettle` | integer | No | Conserva las ventas que tardaron al menos esos dias. Una venta sin liquidar cuenta los dias que lleva abierta. |
| `maxDaysToSettle` | integer | No | El mismo limite, por arriba. |
| `includePayments` | boolean | No | Regresar los abonos de cada venta. Default `true`; con `false` la respuesta va mas ligera cuando solo importan las metricas. |
| `includeCustomerSummary` | boolean | No | Regresar el bloque `byCustomer`. Default `true`; cuesta una agregacion sobre todo el conjunto filtrado. |
| `page` | integer | No | Numero de pagina. Default `1`. |
| `pageSize` | integer | No | Tamano de pagina. Default `50`, maximo `200`. |

:::note `branchCode` no es el mismo codigo que en `7.1`
Aqui es la sucursal **de primer nivel** de Kingdee, la que trae la factura — por ejemplo `801` — no el codigo por departamento que filtra `7.1` (`801.10.02`). Una venta a credito es un documento de Kingdee, asi que carga la sucursal de Kingdee.
:::

:::note Un estatus invalido es un error, no una pagina vacia
Un valor fuera de la tabla regresa `400` con la lista de los validos, para que un typo no se lea como "no hay ninguna":

```json
{ "message": "Unknown status: PAGADA. status must be one of: PAID, OVERDUE, PENDING." }
```
:::

:::note
Todos los filtros viajan en el **query string**. Esto es un `GET`: los filtros mandados como cuerpo JSON se ignoran, y la peticion regresa como si no tuviera filtros.
:::

### Respuesta

`summary` y `byCustomer` cubren **todo lo que matcheo el filtro**, no solo la pagina actual.

```json
{
  "page": 1,
  "pageSize": 50,
  "totalRecords": 8957,
  "totalPages": 180,
  "summary": {
    "totalSales": 8957,
    "totalCustomers": 166,
    "totalAmount": 398789624.49,
    "paidAmount": 352466184.35,
    "outstandingBalance": 46323440.14,
    "paidCount": 7957,
    "overdueCount": 1000,
    "pendingCount": 0,
    "overdueBalance": 46323440.14,
    "avgDaysToSettle": 38.70,
    "weightedAvgDaysToSettle": 42.55,
    "maxDaysToSettle": 452,
    "avgDaysToFirstPayment": 33.54,
    "avgDaysToLastPayment": 35.85,
    "avgDaysOutstanding": 150.36,
    "paymentsCount": 10051,
    "paymentsTotal": 351504525.98
  },
  "byCustomer": [
    {
      "customerId": 1738652,
      "customerCode": "AAA2A102159",
      "customer": "WILLY JINCHAO WU LI",
      "creditLimit": 3000000.00,
      "creditUsed": 2811865.00,
      "creditDays": 30,
      "sales": 24,
      "totalAmount": 1802394.00,
      "paidAmount": 1802394.00,
      "outstandingBalance": 0.00,
      "paidCount": 24,
      "overdueCount": 0,
      "pendingCount": 0,
      "overdueBalance": 0.00,
      "avgDaysToSettle": 23.21,
      "weightedAvgDaysToSettle": 22.34,
      "maxDaysToSettle": 31,
      "avgDaysToLastPayment": 22.92,
      "avgDaysOutstanding": null
    }
  ],
  "sales": [
    {
      "invoiceId": 267078,
      "folio": "XSCKD166673",
      "saleFolio": "2605-05331",
      "saleId": 98698,
      "invoiceCode": null,
      "billDate": "2026-05-30T12:44:49.483",
      "customerId": 1738163,
      "customerCode": "MIC3A100158",
      "customer": "QUINTANA CASILLAS IVAN",
      "branchCode": "801",
      "branch": "Mexico",
      "warehouse": "Cedis Ceylan",
      "sellerId": 131,
      "seller": "Juan Osvaldo Perea Ceja",
      "total": 54400.00,
      "paid": 54400.00,
      "balance": 0.00,
      "creditDays": 30,
      "dueDate": "2026-07-07T00:00:00",
      "settledAt": "2026-06-11T16:05:32.33",
      "daysToSettle": 12,
      "isSettled": true,
      "daysRemaining": 21,
      "daysOverdue": 0,
      "status": "PAID",
      "statusRaw": "Pagada",
      "firstPaymentDate": "2026-06-04T00:00:00",
      "lastPaymentDate": "2026-06-04T00:00:00",
      "daysToFirstPayment": 5,
      "daysToLastPayment": 5,
      "paymentsCount": 1,
      "paymentsTotal": 54400.00,
      "payments": [
        {
          "paymentId": 44650,
          "folio": "PAY-0626-000900",
          "amount": 54400.00,
          "paymentAmount": 54400.00,
          "paymentDate": "2026-06-04T00:00:00",
          "appliedDate": "2026-06-11T16:05:32.357",
          "daysFromSale": 5,
          "paymentFormCode": "03",
          "paymentForm": "Transferencia electronica de fondos",
          "bank": "MIFEL",
          "reference": "9069923",
          "paymentType": "credit",
          "statusId": 29,
          "statusRaw": "Valido",
          "status": "VALID"
        }
      ]
    }
  ]
}
```

**Campos de `summary`:**

| Campo | Tipo | Descripcion |
| --- | --- | --- |
| `totalSales` | integer | Ventas a credito que matchearon el filtro. |
| `totalCustomers` | integer | Clientes distintos detras de esas ventas. |
| `totalAmount` | decimal | Suma de `total`: todo lo vendido a credito. |
| `paidAmount` | decimal | Suma de `paid`: cuanto se ha cobrado. |
| `outstandingBalance` | decimal | Suma de `balance`: cuanto se sigue debiendo. |
| `paidCount` | integer | Ventas en `PAID`. |
| `overdueCount` | integer | Ventas en `OVERDUE`. |
| `pendingCount` | integer | Ventas en `PENDING`. |
| `overdueBalance` | decimal | Saldo que aun deben las ventas vencidas. |
| `avgDaysToSettle` | decimal | Promedio de `daysToSettle` de las ventas **liquidadas**. Es `null` cuando ninguna se ha liquidado. |
| `weightedAvgDaysToSettle` | decimal | El mismo promedio ponderado por el importe de cada venta — la cifra de DSO. |
| `maxDaysToSettle` | integer | El `daysToSettle` mas alto entre las liquidadas. |
| `avgDaysToFirstPayment` | decimal | Promedio de dias de la venta a su primer abono. |
| `avgDaysToLastPayment` | decimal | Promedio de dias de la venta a su ultimo abono. |
| `avgDaysOutstanding` | decimal | Promedio de dias que llevan abiertas las ventas **sin liquidar**. Es `null` cuando todas las que matchearon estan liquidadas. |
| `paymentsCount` | integer | Abonos aplicados a las ventas que matchearon. |
| `paymentsTotal` | decimal | Cuanto suman esos abonos. |

`byCustomer[]` repite las mismas cifras por cliente — mas `creditLimit`, `creditUsed` y `creditDays` de su linea de credito — ordenado por saldo pendiente, de mayor a menor.

:::note Un resultado vacio tambien es una respuesta valida
Un filtro que no matchea nada regresa `totalRecords: 0` con `sales` y `byCustomer` vacios y todos los promedios de `summary` en `null`. El endpoint lee `kingdee_sales_invoices`, `PaymentApplications`, `Payments`, `CreditLines` y `credits`; en un ambiente donde `kingdee_sales_invoices` este vacia regresa cero ventas, lo que significa que ahi no hay ventas a credito que reportar — no que el endpoint haya fallado.
:::

## 10. Estado de cuenta del cliente

El estado de cuenta de un cliente: los mismos datos que muestra el ERP en su pantalla del cliente — un resumen con los saldos de la cuenta y los movimientos, cada uno con su cargo, su abono y el saldo corrido despues de el. Cada pago trae ademas su detalle y los recibos de cobro (收款单) que se mandaron a Kingdee.

:::info Idioma
Este endpoint refleja la pantalla del ERP, asi que la ruta y los nombres de los campos del JSON estan en **espanol** — a diferencia de `7`, `8` y `9`, que estan completamente en ingles.
:::

### Como se arma el estado de cuenta

El endpoint lee las mismas fuentes que la pantalla del ERP y su exportacion a Excel, asi que las cifras nunca discrepan de lo que ve el usuario en el ERP:

- **Movimientos:** los renglones de `dbo.vw_customer_ledger` del cliente — ventas de contado y a credito, todos los pagos, devoluciones, ventas POS, saldos a favor de garantias y aplicaciones de cupones — con el saldo corrido ya acumulado en orden cronologico sobre **toda** la historia del cliente. Los filtros `desde`/`hasta` y `buscar` solo acotan lo que se muestra: el saldo de cada renglon es el saldo real de la cuenta en ese momento, no el acumulado del rango filtrado.
- **Filtro de renglones:** igual que la pantalla del ERP y su exportacion a Excel, se omiten los renglones de `Pago (sobrante)`, los renglones en cero (`cargo = 0` y `abono = 0`) y los pagos derivados de otro pago (`Payments.OriginPaymentId <> 0`).
- **Orden:** los mas recientes primero (`fecha DESC, source_id DESC`), el mismo orden de la pantalla del ERP.

La vista marca cada renglon con un `tipo`:

| Tipo | Cargo / abono |
| --- | --- |
| `Venta` (contado) y `Credito` | Cargo por el `bill_total_amount` de `kingdee_sales_invoices`. |
| `Venta POS` | Cargo por el total de la venta POS (`KingDeeSalesPOS`). |
| `Pago` (cuenta, anticipo o credito) | Abono por el monto del pago (`Payments`). |
| `Devolucion` | Cargo por el monto devuelto (`PaymentRefunds`). |
| `Saldo a Favor. (Garantias)` / `Cupon. (Garantias)` | Abono por el saldo a favor o el cupon emitido. |
| `Aplicacion Nota Garantia` / `Aplicacion Nota Garantia SF` | Cargo cuando se aplica un cupon o un saldo a favor. |

### Resumen

El bloque `resumen` trae las mismas tarjetas de la pantalla del ERP, calculadas por el mismo stored procedure (`dbo.spGetKpisCustomer`), sobre toda la historia del cliente (no el rango filtrado):

| Campo | De donde sale |
| --- | --- |
| `totalCompras` | Total facturado al cliente (`kpi_total`). |
| `saldoAFavor` | Saldo a favor disponible: saldos de garantias sin aplicar, sobrantes de pedidos ya facturados, anticipos sin aplicar y cupones vigentes (`kpi_favor`). |
| `creditoUsado` | Credito usado segun `vw_credits` (`kpi_credito_usado`). Cero si el cliente no tiene linea de credito. |
| `creditoDisponible` | Linea de credito menos credito usado (`kpi_credito_dispo`). |
| `lineaCredito` | Linea de credito autorizada (`kpi_credito_linea`). |
| `saldoActual` | Saldo corrido de la cuenta (`kpi_saldo_act`): positivo si el cliente debe, negativo si los pagos superan lo facturado. |

### 10.1 Consultar el estado de cuenta de un cliente

```http
GET http://pfconexionlinkbits.ddns.net:50780/api/estado-cuenta/{customerCode}
```

```http
GET .../api/estado-cuenta/CUST0017
GET .../api/estado-cuenta/CUST0017?desde=2026-05-01&hasta=2026-05-31
GET .../api/estado-cuenta/CUST0017?buscar=XSCKD16&pagina=1&tamanoPagina=10
GET .../api/estado-cuenta/CUST0017?buscar=PAY-0926-004439
```

| Parametro | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `customerCode` | string | Si | Codigo del cliente en la ruta (`customers.customer_code`). Ejemplo: `CUST0017` |
| `desde` | date | No | Limite inferior sobre la fecha del movimiento. Ejemplo: `2026-05-01` |
| `hasta` | date | No | Limite superior sobre la fecha del movimiento. Si se manda sin hora, incluye el dia completo. |
| `buscar` | string | No | Coincidencia parcial sobre el documento, el concepto, el tipo o el estatus. |
| `pagina` | integer | No | Numero de pagina. Default `1`. |
| `tamanoPagina` | integer | No | Tamano de pagina. Default `50`, maximo `200`. |

:::note Un cliente equivocado es un 404
Un `customerCode` que no existe en `customers` regresa `404` con `{ "message": "El cliente {customerCode} no existe." }`.
:::

### Respuesta

```json
{
  "codigoCliente": "CUST0017",
  "cliente": "ITBS S.A. DE C.V.",
  "resumen": {
    "totalCompras": 54400.00,
    "saldoAFavor": 0.00,
    "creditoUsado": 2811865.00,
    "creditoDisponible": 188135.00,
    "lineaCredito": 3000000.00,
    "saldoActual": 0.00
  },
  "pagina": 1,
  "tamanoPagina": 50,
  "totalRegistros": 2,
  "totalPaginas": 1,
  "movimientos": [
    {
      "fecha": "2026-05-30T12:44:49.483",
      "tipo": "Credito",
      "concepto": "Credito",
      "documento": "XSCKD166673",
      "cargo": 54400.00,
      "abono": 0.00,
      "saldo": 54400.00,
      "estatus": "FINALIZADO",
      "pago": null,
      "recibosKingdee": []
    },
    {
      "fecha": "2026-06-04T10:12:00",
      "tipo": "Pago",
      "concepto": "Pago Cuenta.",
      "documento": "PAY-0626-000900",
      "cargo": 0.00,
      "abono": 54400.00,
      "saldo": 0.00,
      "estatus": "Aplicado",
      "pago": { "...": "ver Detalle del pago y de los recibos" },
      "recibosKingdee": []
    }
  ]
}
```

**Campos de `movimientos[]`:**

| Campo | Tipo | Descripcion |
| --- | --- | --- |
| `fecha` | datetime? | Fecha del movimiento; `null` en movimientos sin fecha (p. ej. saldos a favor aplicados). |
| `tipo` | string | Tipo de la vista: `Venta`, `Credito`, `Pago`, `Venta POS`, `Devolucion`, `Saldo a Favor. (Garantias)`, `Cupon. (Garantias)`, `Aplicacion Nota Garantia` o `Aplicacion Nota Garantia SF`. |
| `concepto` | string | Concepto del movimiento (p. ej. `Pago Anticipo.`, `Pago Cuenta.`, `Credito`). |
| `documento` | string | Folio del documento: `bill_code`, folio del pago, ticket POS o cupon. |
| `cargo` | decimal | Importe que carga saldo. Cero en los abonos. |
| `abono` | decimal | Importe que abona saldo. Cero en los cargos. |
| `saldo` | decimal | Saldo de la cuenta despues de este movimiento, en orden cronologico. |
| `estatus` | string | Estatus del movimiento segun la vista. |
| `pago` | object? | Detalle del pago (ver abajo). Solo en movimientos `Pago`; `null` en los demas. |
| `recibosKingdee` | array | Recibos de cobro (收款单) que el ERP mando a Kingdee por este movimiento (ver abajo). Vacio si no se mando ninguno. |

`totalRegistros` y `totalPaginas` cuentan los movimientos que cumplen los filtros, no solo la pagina actual.

### Detalle del pago y de los recibos

Cada movimiento de la pagina trae dos bloques extra. Solo se llenan para los movimientos de la pagina pedida, no para toda la historia.

- **`pago`** — en los movimientos `Pago`: los mismos datos de la pantalla *Informacion del Pago* del ERP (banco, numero de cuenta, CLABE, metodo de pago, referencia, cotizacion, quien lo subio y quien lo valido), mas los sobrantes que genero el pago y las ventas donde se aplico el dinero.
- **`recibosKingdee`** — los recibos de cobro (收款单, `CreateARReceiveBill`) tal como el ERP los mando, leidos de la cola de envios: encabezado, estatus del envio, el folio `SKD...` que regreso Kingdee y cada renglon (`carts[]`).
  - En una **venta** (`Venta`, `Credito`, `Venta POS`): los recibos amarrados al folio de la venta (`verify_no`), con todos sus renglones.
  - En un **pago**: los recibos donde viajo ese pago, con **solo los renglones de ese pago**. Un recibo suele llevar varios pagos, asi que `total` es la suma de los renglones que regresan, no el total del recibo.

:::caution Los sobrantes viajan con su propio folio
Cuando un pago es mayor que la venta, el ERP crea un pago de *sobrante* (`Payments.OriginPaymentId`) con el remanente, y ese sobrante es el que se aplica a la siguiente venta. Sus renglones en Kingdee llevan en `memo` el **folio del sobrante**, no el del pago original. El estado de cuenta no lista los sobrantes como movimientos, asi que vienen dentro de `pago.sobrantes`, y los `recibosKingdee` del pago original ya incluyen los renglones de todos sus sobrantes.
:::

:::note Recibos anteriores a que la cola guardara el documento
Los recibos mas viejos (de antes de que la cola guardara lo que mandaba) llegan con `tieneDetalle: false`: solo se conocen el folio `SKD`, la venta, el estatus y las fechas; los campos del encabezado y los `renglones` llegan vacios.
:::

**Ejemplo** — un movimiento `Pago` de 90,000 cuyo sobrante se aplico a otra venta:

```json
{
  "fecha": "2026-09-21T11:06:46.777",
  "tipo": "Pago",
  "concepto": "Pago Cuenta.",
  "documento": "PAY-0926-004439",
  "cargo": 0.00,
  "abono": 90000.00,
  "saldo": 82427.50,
  "estatus": "Aplicado",
  "pago": {
    "id": 66299,
    "folio": "PAY-0926-004439",
    "monto": 90000.00,
    "fechaPago": "2026-09-19T00:00:00",
    "fechaRegistro": "2026-09-21T11:06:46.777",
    "fechaRevision": "2026-09-21T11:11:55.877",
    "fechaCancelacion": null,
    "tipoPago": "payment",
    "formaPagoCodigo": "01",
    "formaPago": "Deposito en efectivo",
    "formaPagoCodigoKingdee": "JSFS04_SYS",
    "banco": "BBVA",
    "numeroCuenta": "0113216772",
    "clabe": "012320001132167724",
    "cuenta": "MASSIVE HOME SA DE CV",
    "referencia": "000054632",
    "estatus": "Valido",
    "departamento": "Rutas",
    "cotizacion": "2609-02728",
    "cotizacionTotal": 168018.00,
    "subidoPor": "Nombre del vendedor",
    "validadoPor": "Nombre de finanzas",
    "rechazadoPor": null,
    "ultimaActualizacionPor": "Nombre de finanzas",
    "comentarios": "2609-02728 BBVA $90,000.00 (19-09-26)",
    "observaciones": "CONFIRMADO",
    "sobrantes": [
      {
        "id": 66619,
        "folio": "PAY-0926-004759",
        "monto": 45556.00,
        "fechaRegistro": "2026-09-22T10:46:58.063",
        "pagoOrigen": "PAY-0926-004439"
      }
    ],
    "aplicaciones": [
      {
        "folioPago": "PAY-0926-004439",
        "cotizacion": "2609-02728",
        "venta": "XSCKD218937",
        "esPos": false,
        "monto": 38924.00,
        "fecha": "2026-09-22T10:41:53.18"
      },
      {
        "folioPago": "PAY-0926-004759",
        "cotizacion": "2609-02944",
        "venta": "XSCKD218942",
        "esPos": false,
        "monto": 13800.00,
        "fecha": "2026-09-22T10:50:24.383"
      }
    ]
  },
  "recibosKingdee": [
    {
      "folio": "SKD00166573",
      "origen": "Venta",
      "estatusEnvio": "Enviado",
      "error": null,
      "intentos": 1,
      "fechaRegistro": "2026-09-22T10:43:31.453",
      "fechaEnvio": "2026-09-22T10:43:33.203",
      "venta": "XSCKD218937",
      "tieneDetalle": true,
      "fechaDocumento": "2026-09-21",
      "fechaValor": "2026-09-21",
      "organizacion": "801",
      "sucursal": "801.01.01",
      "departamento": "801010102",
      "moneda": "PRE008",
      "codigoCliente": "CUST0017",
      "socio": "CUST0017",
      "nombreSocio": "NOMBRE DEL CLIENTE",
      "comentario": "PAY-0926-004437, PAY-0926-004438, PAY-0926-004439",
      "total": 38924.00,
      "renglones": [
        {
          "folioPago": "PAY-0926-004439",
          "formaLiquidacion": "JSFS04_SYS",
          "cuentaBancaria": "012320001132167724",
          "cuentaEfectivo": null,
          "cuentaInterna": "REGIONES",
          "monto": 38924.00,
          "comision": 0.00,
          "diferencia": 0.00,
          "referencia": "000054632"
        }
      ]
    }
  ]
}
```

**Campos de `pago`:**

| Campo | Tipo | Descripcion |
| --- | --- | --- |
| `id` | integer | Id del pago (`Payments.Id`). |
| `folio` | string | Folio del pago (`PAY-MMYY-NNNNNN`). |
| `monto` | decimal | Monto del pago. |
| `fechaPago` | datetime? | Fecha del deposito o transferencia segun el comprobante. |
| `fechaRegistro` | datetime? | Cuando se subio el pago al ERP. |
| `fechaRevision` | datetime? | Cuando lo reviso finanzas. `null` si todavia no se revisa. |
| `fechaCancelacion` | datetime? | Cuando se cancelo. |
| `tipoPago` | string? | Tipo de pago del ERP (`Payments.PaymentType`). |
| `formaPagoCodigo` | string? | Codigo SAT de la forma de pago. Ejemplo: `01` |
| `formaPago` | string? | Forma de pago. Ejemplo: `Deposito en efectivo` |
| `formaPagoCodigoKingdee` | string? | Codigo de la forma de pago en el recibo de Kingdee (`SettleType_code`) segun el catalogo de **hoy**. Lo que se mando de verdad esta en `recibosKingdee[].renglones[].formaLiquidacion`. |
| `banco` | string? | Banco. |
| `numeroCuenta` | string? | Numero de cuenta del banco. |
| `clabe` | string? | CLABE del banco. |
| `cuenta` | string? | Razon social que recibe el dinero. Ejemplo: `MASSIVE HOME SA DE CV` |
| `referencia` | string? | Referencia bancaria. |
| `estatus` | string? | Estatus del pago en el ERP. Ejemplo: `Valido` |
| `departamento` | string? | Departamento del pago. |
| `cotizacion` | string? | Folio de la cotizacion a la que se subio el pago. Ejemplo: `2609-02728` |
| `cotizacionTotal` | decimal? | Total de esa cotizacion. |
| `subidoPor` / `validadoPor` / `rechazadoPor` / `ultimaActualizacionPor` | string? | Usuarios que subieron, validaron, rechazaron y actualizaron por ultima vez el pago. |
| `comentarios` / `observaciones` | string? | Texto libre capturado en el pago. |
| `sobrantes[]` | array | Pagos de sobrante que nacieron de este, de todos los niveles de la cadena: `id`, `folio`, `monto`, `fechaRegistro` y `pagoOrigen` (folio del pago del que nacio). |
| `aplicaciones[]` | array | Ventas donde se aplico el dinero, de este pago y de sus sobrantes: `folioPago` (el pago o sobrante que se aplico), `cotizacion`, `venta` (`XSCKD...` o ticket POS; `null` si la venta aun no esta en Kingdee), `esPos`, `monto` y `fecha`. |

**Campos de `recibosKingdee[]`:**

| Campo | Tipo | Descripcion |
| --- | --- | --- |
| `folio` | string? | Folio del recibo en Kingdee (`SKD...`). `null` si Kingdee no lo creo. |
| `origen` | string | `Venta` (cobro de una cotizacion), `Venta POS` (pagos vinculados a un ticket de mostrador) o `Credito` (abono a una venta a credito). |
| `estatusEnvio` | string | `Pendiente`, `Procesando`, `Enviado`, `Fallido`, `Inconsistente` u `Omitido` (la venta no tenia pagos que mandar). |
| `error` | string? | Ultimo mensaje de rechazo de Kingdee, si lo hubo. |
| `intentos` | integer | Intentos de envio. |
| `fechaRegistro` | datetime? | Cuando el recibo entro a la cola. |
| `fechaEnvio` | datetime? | Cuando Kingdee lo acepto. `null` si `estatusEnvio` no es `Enviado`. |
| `venta` | string? | Venta a la que se amarra el recibo (`verify_no`). |
| `tieneDetalle` | boolean | `false` en recibos mandados antes de que la cola guardara el documento; los campos de abajo llegan vacios. |
| `fechaDocumento` | string? | `bill_date`. |
| `fechaValor` | string? | `value_date`. |
| `organizacion` | string? | `branch_code`. Ejemplo: `801` |
| `sucursal` | string? | `settle_branch_code`. Ejemplo: `801.01.01` |
| `departamento` | string? | `dept_code`. |
| `moneda` | string? | `currency_code`. |
| `codigoCliente` | string? | `customer_code`: el cliente, o la sucursal en las ventas de tienda. |
| `socio` | string? | `member_card_no`: el codigo del cliente del ERP. |
| `nombreSocio` | string? | `member_name`. |
| `comentario` | string? | `remark`: folios de los pagos del recibo. |
| `total` | decimal | Suma de los renglones que regresan. |
| `renglones[]` | array | Renglones (`carts[]`) del recibo, ver abajo. |

**Campos de `recibosKingdee[].renglones[]`:**

| Campo | Campo en Kingdee | Descripcion |
| --- | --- | --- |
| `folioPago` | `memo` | Folio del pago aplicado (o de su sobrante). |
| `formaLiquidacion` | `SettleType_code` | Forma de liquidacion. Ejemplo: `JSFS04_SYS` |
| `cuentaBancaria` | `receive_bank_account` | CLABE, o la cuenta fija de la sucursal. Vacia en efectivo. |
| `cuentaEfectivo` | `receive_cash_account` | Cuenta de efectivo del departamento. Vacia cuando hay banco. |
| `cuentaInterna` | `inner_account_no` | Cuenta interna del departamento. |
| `monto` | `Amount` | Monto del renglon. |
| `comision` | `handling_charge_fee` | Comision. |
| `diferencia` | `over_under_amount` | Diferencia. |
| `referencia` | `reference` | Referencia bancaria del pago. |

## 11. Clientes (endpoint en ingles)

Los datos basicos de un cliente — contacto, empresa, categoria, sucursal y datos fiscales — junto con las direcciones que tiene registradas.

:::info Idioma
Esta API esta completamente **en ingles** — rutas y nombres de campos — igual que `7`, `8` y `9`.
:::

### Las dos llaves de un cliente

La tabla `customers` tiene dos llaves distintas, y la respuesta expone ambas:

| Campo | Columna | Para que sirve |
| --- | --- | --- |
| `id` | `customers.id` | Llave primaria del renglon del cliente. Las direcciones (`customer_addresses.customer_id`) apuntan a esta. |
| `customerId` | `customers.customer_id` | El id con el que se une el resto del ERP: ventas, pagos y estado de cuenta. Es `null` en clientes que nunca lo recibieron. |

:::caution Las direcciones no cuelgan de `customerId`
A pesar del nombre de la columna, `customer_addresses.customer_id` apunta a `customers.id`, no a `customers.customer_id`. Usa `id` para relacionar una direccion con su cliente, y `customerCode` o `customerId` para relacionar al cliente con el resto de las APIs.
:::

### Estatus y categoria

| Campo | De donde sale |
| --- | --- |
| `statusId` / `status` | `customers.statusId` → `catEstatus`. Valores en uso: `1` (`ACTIVO`), `2` (`ELIMINADO`), `8` (`CANCELADO`). |
| `categoryCode` / `category` | `customers.customer_categoryId` guarda el **codigo** de `Customer_categories` (`C3`, `C5`, `C9`, …), no su id. Ejemplo: `C5` → `MIEMBRO`. |
| `branchId` / `branch` | Sucursal del cliente, tal como esta guardada en el renglon del cliente. |

### 11.1 Listar clientes

```http
GET https://bamboonetapi.ddns.net/api/customers
```

```http
GET .../api/customers?search=Martinez&onlyActive=true
GET .../api/customers?categoryCode=C9&page=1&pageSize=20
GET .../api/customers?city=Monterrey&includeAddresses=false
```

| Parametro | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `search` | string | No | Coincidencia parcial sobre el codigo, nombre, razon social, email, telefono (`phone` o `phoneAlt`) o RFC. |
| `customerCode` | string | No | Codigo exacto del cliente. Es unico en el catalogo. |
| `statusId` | integer | No | Estatus del cliente: `1` (ACTIVO), `2` (ELIMINADO) u `8` (CANCELADO). |
| `onlyActive` | boolean | No | Atajo de `statusId=1`. Se ignora cuando se manda `statusId`. |
| `categoryCode` | string | No | Codigo de categoria. Ejemplo: `C5` |
| `branchId` | integer | No | Sucursal del cliente. |
| `city` | string | No | Coincidencia parcial sobre la ciudad del cliente. |
| `state` | string | No | Coincidencia parcial sobre el estado del cliente. |
| `startDate` | date | No | Clientes dados de alta en o despues de este dia (`createdAt`). |
| `endDate` | date | No | Clientes dados de alta en o antes de este dia. Si se manda sin hora, incluye el dia completo. |
| `includeAddresses` | boolean | No | Trae las direcciones de cada cliente de la pagina. Default `true`; manda `false` para omitirlas. |
| `page` | integer | No | Numero de pagina. Default `1`. |
| `pageSize` | integer | No | Tamano de pagina. Default `50`, maximo `200`. |

Los clientes se ordenan por nombre.

### Respuesta

```json
{
  "page": 1,
  "pageSize": 50,
  "totalRecords": 18,
  "totalPages": 1,
  "customers": [
    {
      "id": 1024,
      "customerId": 500123,
      "customerCode": "CUST0042",
      "name": "Laura Martinez Rios",
      "statusId": 1,
      "status": "ACTIVO",
      "phone": "8112345678",
      "phoneAlt": null,
      "email": "cliente@ejemplo.com",
      "address": "Av. Constitucion 100",
      "country": "Mexico",
      "state": "Nuevo León",
      "city": "Monterrey",
      "birthDate": "1990-01-01T00:00:00",
      "gender": "F",
      "companyName": null,
      "companyType": null,
      "companySize": null,
      "categoryCode": "C5",
      "category": "MIEMBRO",
      "branchId": 100,
      "branch": "Sucursal Monterrey",
      "assignedUserId": null,
      "createdAt": "2026-06-16T13:40:19.583",
      "updatedAt": "2026-06-16T13:40:19.583",
      "tax": {
        "invoiceRequired": true,
        "taxId": "XAXX010101000",
        "name": "LAURA MARTINEZ RIOS",
        "email": "facturas@ejemplo.com",
        "address": "Av. Constitucion 100",
        "zipCode": "64000",
        "regimeCode": "612",
        "useCode": "G03",
        "personType": "Fisica"
      },
      "addresses": [
        {
          "id": 2048,
          "addressType": "Punto de Venta",
          "street": "Av. Constitucion 100",
          "neighborhood": "Centro",
          "postalCode": "64000",
          "city": "Monterrey",
          "state": "Nuevo León",
          "country": "Mexico",
          "createdAt": "2026-06-16T13:40:19.583",
          "updatedAt": null
        }
      ]
    }
  ]
}
```

**Campos de `customers[]`:**

| Campo | Tipo | Descripcion |
| --- | --- | --- |
| `id` | integer | Llave primaria del renglon del cliente (`customers.id`). |
| `customerId` | integer? | Id del cliente que usa el resto del ERP (`customers.customer_id`). |
| `customerCode` | string | Codigo del cliente. |
| `name` | string | Nombre del cliente. |
| `statusId` / `status` | integer / string | Estatus del cliente y su nombre. |
| `phone` / `phoneAlt` | string | Telefono principal y alterno. |
| `email` | string | Email del cliente. |
| `address` | string | Direccion en texto libre guardada en el propio renglon del cliente. Es independiente de `addresses[]`. |
| `country` / `state` / `city` | string | Ubicacion del cliente. |
| `birthDate` / `gender` | date? / string | Datos personales, cuando se capturaron. |
| `companyName` / `companyType` / `companySize` | string | Datos de la empresa, cuando el cliente es empresa. |
| `categoryCode` / `category` | string | Codigo y nombre de la categoria. |
| `branchId` / `branch` | integer? / string | Sucursal del cliente. |
| `assignedUserId` | integer? | Vendedor al que esta asignado el cliente. |
| `createdAt` / `updatedAt` | datetime | Alta y ultima actualizacion del cliente. |
| `tax` | object | Datos fiscales para facturar al cliente (ver abajo). |
| `addresses` | array | Direcciones del cliente (ver abajo). Vacio cuando no tiene o cuando `includeAddresses=false`. |

**Campos de `tax`:**

| Campo | Tipo | Descripcion |
| --- | --- | --- |
| `invoiceRequired` | boolean? | Si el cliente pide factura. |
| `taxId` | string | RFC del cliente. |
| `name` | string | Razon social para facturar. |
| `email` | string | Email al que se envian las facturas. |
| `address` | string | Domicilio fiscal. |
| `zipCode` | string | Codigo postal fiscal. |
| `regimeCode` | string | Clave de regimen fiscal del SAT. Ejemplo: `612` |
| `useCode` | string | Clave de uso de CFDI del SAT. Ejemplo: `G03` |
| `personType` | string | Si el cliente es persona fisica o moral. |

**Campos de `addresses[]`:**

| Campo | Tipo | Descripcion |
| --- | --- | --- |
| `id` | integer | Id de la direccion. |
| `addressType` | string | Tipo de direccion: `Punto de Venta`, `Envío a domicilio`, `Facturación` o `POS`. |
| `street` | string | Calle y numero. |
| `neighborhood` | string | Colonia. Puede ser `null`. |
| `postalCode` | string | Codigo postal. |
| `city` / `state` / `country` | string | Ubicacion de la direccion. |
| `createdAt` / `updatedAt` | datetime? | Alta y ultima actualizacion de la direccion. |

:::note Compara `addressType` de forma laxa
`addressType` se regresa tal cual esta guardado en el ERP. Algunos renglones viejos traen los acentos doblemente codificados (por ejemplo, una variante rota de `Envío a domicilio`), asi que no dependas de los acentos exactos al comparar este campo.
:::

`totalRecords` y `totalPages` cuentan los clientes que coinciden con los filtros, no solo la pagina actual.

### 11.2 Consultar un cliente

```http
GET https://bamboonetapi.ddns.net/api/customers/{customerCode}
```

```http
GET .../api/customers/CUST0042
```

| Parametro | Tipo | Requerido | Descripcion |
| --- | --- | --- | --- |
| `customerCode` | string | Si | Codigo del cliente en la ruta (`customers.customer_code`). Ejemplo: `CUST0042` |

Regresa un solo cliente con la misma forma que un elemento de `customers[]` en `11.1` — sin el envoltorio de paginacion — y siempre con sus direcciones.

:::note Un cliente inexistente es un 404
Un `customerCode` que no existe en `customers` regresa `404` con `{ "message": "El cliente {customerCode} no existe." }`.
:::
