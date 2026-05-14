---
sidebar_position: 2
title: Authentication
---

# Authentication

All APIs require an API Key. This key identifies the integration querying the information.

:::warning Important
The API Key must be kept private. Do not include it in public web applications, repositories, logs, screenshots, or URLs.
:::

## Required header

Include the API Key in every request:

```http
X-API-Key: YOUR_API_KEY_HERE
Accept: application/json
```

## curl example

```bash
curl -X GET "https://bamboonetapi.ddns.net/api/pedidos/2605-00005" \
  -H "X-API-Key: YOUR_API_KEY_HERE" \
  -H "Accept: application/json"
```

## JavaScript example

```javascript
async function queryOrder(folio) {
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
    throw new Error(`HTTP error ${response.status}`);
  }

  return response.json();
}
```

## Authentication responses

| Code | Meaning | What to check |
| --- | --- | --- |
| `200` | Authorized request | The API Key is valid |
| `401` | Unauthorized | Missing `X-API-Key` or invalid key |

## Recommendations

- Store the API Key in environment variables or a secrets manager.
- Consume these APIs from a backend or secure service.
