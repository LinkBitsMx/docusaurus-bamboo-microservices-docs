---
sidebar_position: 4
title: Ejemplos de integracion
---

# Ejemplos de integracion

## Consultar envio desde JavaScript

```javascript
async function consultarEnvio(folio) {
  const response = await fetch(
    `https://bamboonetapi.ddns.net/api/envios/${encodeURIComponent(folio)}`,
    {
      method: 'GET',
      headers: {
        'X-API-Key': 'TU_API_KEY',
        'Accept': 'application/json',
      },
    }
  );

  if (response.status === 404) {
    return {
      encontrado: false,
      mensaje: 'No encontramos informacion para ese folio.',
    };
  }

  if (!response.ok) {
    throw new Error(`Error HTTP ${response.status}`);
  }

  const pedidos = await response.json();

  return {
    encontrado: true,
    pedidos: pedidos.map((pedido) => ({
      pedidoId: pedido.pedidoId,
      paqueteria: pedido.paqueteria,
      estatusEnvio: pedido.estatusEnvio,
      fechaPedido: pedido.fechaPedido,
      guias: pedido.guias,
    })),
  };
}
```

## Mensaje sugerido para el usuario final

Cuando el folio contiene varios pedidos internos:

```text
Informacion de envio encontrada para el folio 2509-03200:

Pedido 25086
Paqueteria: FLETERA
Guia: Sin guia

Pedido 25087
Paqueteria: PAQUETEXPRESS
Guias:
- MEX01PP3469501006006
- MEX01PP3469501006005
- MEX01PP3469501006004
- MEX01PP3469501006003
- MEX01PP3469501006002
- MEX01PP3469501006001
```

Cuando uno de los pedidos tiene guia:

```text
Tu pedido ya cuenta con guia.
Paqueteria: PAQUETEXPRESS
Guia: MEX01PP3469501006006
Rastreo: https://www.paquetexpress.com.mx/rastreo/MEX01PP3469501006006
```

Cuando uno de los pedidos tiene paqueteria sin guia:

```text
Tu pedido ya tiene paqueteria asignada: FLETERA.
La guia aun no esta disponible. Puedes volver a consultar mas tarde.
```

Cuando no existe informacion:

```text
No encontramos informacion para ese folio. Revisa que este escrito correctamente.
```

## Consultar una venta con su estatus por almacen desde JavaScript

```javascript
async function consultarVenta(folio) {
  const response = await fetch(
    `https://bamboonetapi.ddns.net/api/sales/${encodeURIComponent(folio)}`,
    {
      method: 'GET',
      headers: {
        'X-API-Key': 'TU_API_KEY_AQUI',
        'Accept': 'application/json',
      },
    }
  );

  if (response.status === 404) {
    return {
      encontrado: false,
      mensaje: 'No encontramos una venta para ese folio.',
    };
  }

  if (!response.ok) {
    throw new Error(`Error HTTP ${response.status}`);
  }

  const venta = await response.json();

  return {
    encontrado: true,
    folio: venta.folio,
    total: venta.totals.total,
    // Mientras la venta esta en cotizacion hay un solo estatus y aun no hay almacenes.
    esCotizacion: venta.status.isQuotation,
    estatus: venta.status.status,
    almacenes: venta.warehouses.map((almacen) => ({
      almacen: almacen.warehouse,
      estatus: almacen.status,
      piezas: almacen.units,
      importe: almacen.amount,
    })),
  };
}
```

Para listar las ventas de un periodo, usa `/api/sales` con `startDate`, `endDate` y paginacion:

```javascript
async function listarVentas({ startDate, endDate, page = 1, pageSize = 50 }) {
  const query = new URLSearchParams({ startDate, endDate, page, pageSize });

  const response = await fetch(
    `https://bamboonetapi.ddns.net/api/sales?${query}`,
    {
      method: 'GET',
      headers: {
        'X-API-Key': 'TU_API_KEY_AQUI',
        'Accept': 'application/json',
      },
    }
  );

  if (!response.ok) {
    throw new Error(`Error HTTP ${response.status}`);
  }

  // totalPages indica si hay que pedir otra pagina.
  return response.json();
}
```

## Mensaje sugerido para una venta dividida

```text
Venta 2607-00037 — total $130,997.00

Cedis Vallejo: empacado en proceso (5,400 piezas)
Cedis Motevideo: guia de envio en proceso (2,200 piezas)
```

Cuando la venta sigue en cotizacion:

```text
La venta 2608-00004 sigue en cotizacion.
Todavia no se asigna a ningun almacen.
```

## Manejo recomendado de errores

| Codigo | Manejo sugerido |
| --- | --- |
| `401` | Revisar API Key. No mostrar detalles tecnicos al usuario final. |
| `404` | Indicar que el folio no fue encontrado. |
| `500` | Pedir al usuario intentar mas tarde o levantar reporte interno. |
