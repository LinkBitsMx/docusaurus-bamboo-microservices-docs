---
sidebar_position: 5
title: Errores y preguntas frecuentes
---

# Errores y preguntas frecuentes

## Codigos de respuesta

| Codigo | Significado | Respuesta recomendada |
| --- | --- | --- |
| `200` | Consulta exitosa | Procesar la respuesta JSON |
| `401` | API Key ausente o invalida | Revisar credenciales |
| `404` | Recurso no encontrado | Validar folio, SKU o ticket |
| `500` | Error interno | Reportar a soporte con datos de la peticion |

## Buenas practicas

- No compartas la API Key en canales publicos.
- No guardes la API Key en repositorios.
- No envies la API Key en la URL.
- Usa HTTPS en ambientes productivos.
- Registra internamente el folio consultado, endpoint, fecha, hora y codigo HTTP.

## Preguntas frecuentes

### Donde obtengo la API Key?

La proporciona el equipo responsable de la integracion.

### La API Key se puede usar desde frontend?

No se recomienda. Debe usarse desde un backend, servicio privado o servidor controlado por el integrador.

### Por que un envio puede tener paqueteria pero no guia?

Porque la paqueteria puede asignarse desde el inicio del pedido, mientras que la guia se genera mas adelante en el proceso operativo.

En ese caso el pedido sigue apareciendo en el arreglo `guias` con `guia: "Sin guia"` y `trackingUrl: null`:

```json
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
}
```

### Por que `/api/envios/{folio}` regresa un arreglo?

Un mismo folio puede dividirse en varios pedidos internos. La API regresa un objeto por cada `pedidoId` para que las integraciones puedan mostrar cada pedido interno por separado.

Si un pedido interno tiene varias guias, esas guias se agrupan dentro del mismo arreglo `guias` en lugar de repetir el pedido.

### Que significa el desglose de stock en un item de pre-orden?

Cada item de una pre-orden se enriquece con el stock entregable (`deliverable_qty`) de cada **almacen de venta** (`sales_enabled = 1`). Con eso la API calcula cuanto de lo pedido se puede cubrir (`cantidadCubierta` / `coveredQuantity`), cuanto falta (`cantidadAgotada` / `shortageQuantity`), una sugerencia de reparto por almacen y un estado de surtido:

- `CUBIERTA` / `COVERED` — un solo almacen puede cubrir toda la cantidad.
- `DISTRIBUIR` / `DISTRIBUTE` — hay stock total suficiente, pero repartido en varios almacenes.
- `AGOTADO_PARCIAL` / `PARTIALLY_COVERED` — hay algo de stock, pero no alcanza para toda la cantidad.
- `SIN_STOCK` / `OUT_OF_STOCK` — ningun almacen de venta tiene existencia.

### Por que `/api/preorders/detail` esta en ingles y los demas endpoints de pre-orden en espanol?

Ese endpoint (ruta, nombres de campos y valores de estado) se mantiene en ingles porque lo revisa el equipo de China. Regresa la misma informacion que `GET /api/PreOrdenes/{id}`, solo que traducida. Los demas endpoints conservan sus campos originales en espanol por compatibilidad.

La API de ventas (`/api/sales`) sigue la misma regla: esta completamente en ingles porque la maneja el equipo de China.

### Por que una venta tiene un estatus y sus almacenes otro?

Por como funciona el proceso operativo. La venta se guarda en `quotation` con un **estatus general**; mientras sigue en cotizacion ese es el unico estatus que existe, y `isQuotation` regresa `true` con `warehouses[]` vacio.

Al validarse la cotizacion, la orden **se divide y cada almacen guarda su propio estatus**: un almacen puede ir ya en guias de envio (`IN_SHIPPING_LABEL`) mientras otro sigue empacando (`IN_PACKING_OR_REVIEW`). A partir de ahi `isQuotation` es `false` y el estatus vigente de cada almacen esta en `warehouses[]`, ademas de en cada item por medio de `warehouseStatus`.

### Cual es la diferencia entre `status` y `statusRaw` en la API de ventas?

`statusRaw` es el nombre del estatus tal cual lo guarda BambooERP (en espanol, por ejemplo `Pago Validado`), util para rastrear el valor original. `status` es ese mismo valor normalizado a las etapas en ingles que publica la API: `IN_QUOTATION`, `SENT_TO_CEDIS`, `IN_PICKING`, `IN_PACKING_OR_REVIEW`, `IN_SHIPPING_LABEL`, `DELIVERED`, `CANCELLED` o `UNKNOWN`.

Las integraciones deben apoyarse en `status`; `statusRaw` es informativo y puede sumar valores nuevos conforme evolucione BambooERP.

### Por que `paymentsTotal` no coincide con el `total` de la venta?

Porque responden preguntas distintas. `total` es el monto con el que cierra la venta (columna `quotation.total`), mientras que `paymentsTotal` es la suma de los pagos realmente registrados contra ella (`payments[].amount`).

Pueden diferir de forma legitima en ambos sentidos: una venta puede quedar parcialmente pagada (total 36,750 con 36,250.40 registrados) o traer pagos por encima de su total. **No uses `paymentsTotal` para dar por liquidada una venta** — revisa los pagos individuales y su `status`.

### Una venta puede tener varios pagos con formas distintas?

Si, y es comun. Una misma venta puede combinar una transferencia, varios saldos a favor y efectivo. Cada elemento de `payments[]` trae su propio monto, forma (`paymentFormCode` / `paymentForm`) y estatus, asi que la mezcla de pagos se puede reconstruir agrupando por forma.

El `paymentFormCode` es el codigo SAT de `sat_FormaPago`: `01` efectivo, `02` cheque, `03` transferencia electronica, `04` tarjeta de credito, `17` saldo a favor, `28` tarjeta de debito, entre otros.

### Por que algunos items de una venta no traen almacen?

Porque son renglones de servicio (`isService: true`), como `ENVIO %` o la fletera. No los surte ningun almacen, asi que regresan con `warehouseId: null` y `warehouseStatus: null`, y se suman en `servicesTotal` en lugar de `productsSubtotal`.
