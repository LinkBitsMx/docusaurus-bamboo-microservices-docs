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
| Consulta de envio | Paqueteria, guia y liga de rastreo cuando existan |
| Precios de producto | Precio por codigo interno o SKU, separado por sucursal |
| Consulta de garantia | Productos y estatus asociados a un folio de garantia |

## Base URL

```text
https://bamboonetapi.ddns.net/api/
```

## Formato general

- Todas las peticiones usan `GET`.
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
GET /pedidos/{folio}
GET /pedidos/{folio}/estatus
GET /envios/{folio}
GET /precios/productos/{identificador}
GET /garantias/{folioTicket}
```
