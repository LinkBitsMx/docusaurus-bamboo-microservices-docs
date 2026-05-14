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

En ese caso la API puede responder:

```json
{
  "paqueteria": "PAQUETEXPRESS",
  "guia": "Sin guia",
  "trackingUrl": null
}
```
