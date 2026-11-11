---
sidebar_position: 1
title: Introduccion
slug: /apis-consulta
---

# APIs de consulta

Esta seccion documenta las APIs de consulta disponibles para integraciones externas.

Estas APIs permiten consultar informacion de BambooERP sin entrar al sistema interno. Estan pensadas para escenarios donde un bot, una integracion o un canal externo necesita responder preguntas comunes sobre pedidos, envios, precios y garantias.

## Que puedes consultar

| API | Que resuelve |
| --- | --- |
| Consulta de pedido | Informacion general de un pedido por folio |
| Estatus de pedido | Avance actual del pedido en el proceso operativo |
| Consulta de envio | Pedidos internos del folio, con paqueteria y guias agrupadas |
| Precios de producto | Precio por codigo interno o SKU, separado por sucursal, junto con la descripcion del producto |
| Consulta de garantia | Productos y estatus asociados a un folio de garantia |
| Pre-ordenes | Enviar solicitudes de cotizacion sin confirmar, con su metodo de entrega, domicilio y formas de pago, y revisarlas con detalle de stock por almacen |
| Ventas | Detalle, totales, vendedor, pagos y estatus por almacen de las ventas registradas en BambooERP |
| Pagos | Registrar un pago en BambooERP, opcionalmente aplicado a una venta y con los campos del documento de Kingdee, y listar los pagos registrados por estatus |
| Ventas a credito | Las ventas que se fueron a credito, cada una con los abonos aplicados, mas las metricas de cobranza del conjunto filtrado |
| Estado de cuenta del cliente | Saldos de la cuenta del cliente y sus movimientos (cargos, abonos y saldo corrido), con el detalle de cada pago y los recibos enviados a Kingdee |
| Clientes | Datos basicos del cliente (contacto, empresa, categoria, sucursal y datos fiscales) junto con sus direcciones registradas |
| Envios | Paqueterias, peso y volumen de productos y el calculo del envio con las mismas reglas con las que cobra BambooERP |
| Vendedores | Vendedores y los clientes que tienen asignados |
| Compra Anticipada | Crear pre-ordenes que vende una sucursal y surte un CEDIS (el cliente recoge en el CEDIS), con almacen por renglon y numero de pedido de la Tienda Online; seguir su surtido, validacion y estatus en Kingdee, y recibir avisos de estatus en la Tienda Online |

## Base URL

```text
https://bamboonetapi.ddns.net/api/
```

## Formato general

- La mayoria de las peticiones usan `GET`; crear una pre-orden, registrar un pago y cotizar un envio usan `POST`.
- Todas las respuestas se entregan en JSON.
- Todas las peticiones requieren el header `X-API-Key`.
- Si un dato aun no existe en BambooERP, la API puede regresar valores como `Sin guia`, `No asignada` o `null`.

## Flujo recomendado

1. Solicita tu API Key al equipo de integracion.
2. Configura la API Key en tu backend o servicio privado.
3. Prueba primero con folios reales de validacion.
4. Maneja correctamente respuestas `401`, `404` y `500`.
5. No expongas la API Key en frontend publico, repositorios o capturas.

## Endpoints

```text
GET  /pedidos/{folio}
GET  /pedidos/{folio}/estatus
GET  /envios/{folio}
GET  /precios/productos/{identificador}
GET  /garantias/{folioTicket}
POST /PreOrdenes
GET  /PreOrdenes
GET  /PreOrdenes/{id}
GET  /PreOrdenes/opciones-entrega
GET  /preorders/detail
GET  /sales
GET  /sales/{folio}
POST /payments
GET  /payments
GET  /payments/{id}
GET  /customers
GET  /customers/{customerCode}
GET  /shipping/carriers
GET  /shipping/products
POST /shipping/quote
GET  /sellers
GET  /sellers/customers
GET  /compra-anticipada/cedis
POST /compra-anticipada/preordenes
GET  /compra-anticipada/preordenes
GET  /compra-anticipada/preordenes/{orderId}
```
