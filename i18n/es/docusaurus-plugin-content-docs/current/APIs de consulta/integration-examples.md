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

  const data = await response.json();

  return {
    encontrado: true,
    pedidoId: data.pedidoId,
    paqueteria: data.paqueteria,
    guia: data.guia,
    trackingUrl: data.trackingUrl,
  };
}
```

## Mensaje sugerido para el usuario final

Cuando la API devuelve guia:

```text
Tu pedido ya cuenta con guia.
Paqueteria: PAQUETEXPRESS
Guia: MEX14PP0067946003003
Rastreo: https://www.paquetexpress.com.mx/rastreo/MEX14PP0067946003003
```

Cuando la API devuelve paqueteria sin guia:

```text
Tu pedido ya tiene paqueteria asignada: PAQUETEXPRESS.
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
