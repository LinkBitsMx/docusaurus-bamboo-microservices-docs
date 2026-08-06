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
  }
}
```

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
