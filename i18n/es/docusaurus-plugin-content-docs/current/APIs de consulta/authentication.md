---
sidebar_position: 2
title: Autenticacion
---

# Autenticacion

Todas las APIs requieren una API Key. Esta clave identifica a la integracion que esta consultando la informacion.

:::warning Importante
La API Key debe mantenerse privada. No la pongas en aplicaciones web publicas, repositorios, logs, screenshots o URLs.
:::

## Header requerido

Incluye la API Key en todas las peticiones:

```http
X-API-Key: YOUR_API_KEY_HERE
Accept: application/json
```

## Ejemplo con curl

```bash
curl -X GET "https://bamboonetapi.ddns.net/api/pedidos/2605-00005" \
  -H "X-API-Key: YOUR_API_KEY_HERE" \
  -H "Accept: application/json"
```

## Ejemplo con JavaScript

```javascript
async function consultarPedido(folio) {
  const response = await fetch(
    `https://bamboonetapi.ddns.net/api/pedidos/${encodeURIComponent(folio)}`,
    {
      method: 'GET',
      headers: {
        'X-API-Key': 'YOUR_API_KEY_HERE',
        'Accept': 'application/json',
      },
    }
  );

  if (!response.ok) {
    throw new Error(`Error HTTP ${response.status}`);
  }

  return response.json();
}
```

## Respuestas de autenticacion

| Codigo | Significado | Que revisar |
| --- | --- | --- |
| `200` | Peticion autorizada | La API Key es valida |
| `401` | No autorizado | Falta `X-API-Key` o la clave es incorrecta |

## Recomendaciones

- Guarda la API Key en variables de entorno o en un gestor de secretos.
- Consume estas APIs desde un backend o servicio seguro.
