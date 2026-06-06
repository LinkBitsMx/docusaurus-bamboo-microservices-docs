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

## Manejo recomendado de errores

| Codigo | Manejo sugerido |
| --- | --- |
| `401` | Revisar API Key. No mostrar detalles tecnicos al usuario final. |
| `404` | Indicar que el folio no fue encontrado. |
| `500` | Pedir al usuario intentar mas tarde o levantar reporte interno. |
