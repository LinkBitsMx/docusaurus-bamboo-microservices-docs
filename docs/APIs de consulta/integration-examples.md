---
sidebar_position: 4
title: Integration examples
---

# Integration examples

## Query shipment from JavaScript

```javascript
async function queryShipment(folio) {
  const response = await fetch(
    `https://bamboonetapi.ddns.net/api/envios/${encodeURIComponent(folio)}`,
    {
      method: 'GET',
      headers: {
        'X-API-Key': 'YOUR_API_KEY_HERE',
        'Accept': 'application/json',
      },
    }
  );

  if (response.status === 404) {
    return {
      found: false,
      message: 'No information was found for that folio.',
    };
  }

  if (!response.ok) {
    throw new Error(`HTTP error ${response.status}`);
  }

  const data = await response.json();

  return {
    found: true,
    pedidoId: data.pedidoId,
    paqueteria: data.paqueteria,
    guia: data.guia,
    trackingUrl: data.trackingUrl,
  };
}
```

## Suggested message for the end user

When the API returns a tracking number:

```text
Your order already has a tracking number.
Carrier: PAQUETEXPRESS
Tracking number: MEX14PP0067946003003
Tracking URL: https://www.paquetexpress.com.mx/rastreo/MEX14PP0067946003003
```

When the API returns a carrier without a tracking number:

```text
Your order already has an assigned carrier: PAQUETEXPRESS.
The tracking number is not available yet. Please check again later.
```

When no information exists:

```text
No information was found for that folio. Please verify that it is written correctly.
```

## Recommended error handling

| Code | Suggested handling |
| --- | --- |
| `401` | Check the API Key. Do not show technical details to the end user. |
| `404` | Indicate that the folio was not found. |
| `500` | Ask the user to try again later or create an internal report. |
