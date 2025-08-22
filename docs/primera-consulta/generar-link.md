---
sidebar_position: 1
---

# Genera un link de cobro

La integración principal de Venepagos se realiza a través de la creación de enlaces de pago. El flujo recomendado es:

1. **Tu backend crea un enlace de pago usando el endpoint `/api/public/payment-links/create`**
2. **Recibes la URL del enlace generado**
3. **Rediriges al usuario a esa URL (en una ventana nueva, WebView, o similar)**
4. **El usuario realiza el pago en la página de Venepagos**
5. **Recibes notificaciones de eventos (pagos completados, fallidos, cancelados) vía Webhook**

---



### Crear un enlace de pago - POST

:::warning

Crea un archivo `.env` para almacenar la `BASE_URL` y `API_KEY` para manetener tus credenciales **seguras** ejemplo.

```js title=".env"
    BASE_URL="https://venepagos.com.ve"
    API_KEY=tu_api_key
```
:::

- **Endpoint:** `/api/public/payment-links/create`
- **Método:** `POST`
- **Headers:**
  - `Authorization: Bearer process.env.API_KEY`
  - `Content-Type: application/json`

El campo `metadata` es un objeto personalizable que te permite incluir datos adicionales que necesites para tu aplicación. Estos datos serán devueltos en las notificaciones de webhook, permitiéndote mantener el contexto de cada pago (por ejemplo: IDs de usuario, orden, producto, etc).

El campo `amount` debe ser el monto del cobro. Si `currency` es **USD**, Venepagos convertirá automáticamente a Bolívares según la tasa del Banco Central de Venezuela **(BCV)**. Si `currency` es **Bs**, se cobrará exactamente ese monto en Bolívares, sin consultar tasa.

El campo `currency` acepta `"USD"` o `"Bs"`. Usa `"Bs"` para cobrar un monto exacto en bolívares.

#### Ejemplo de Request
```json
{
    "title": "Pago de prueba",
    "description": "Pago de producto X",
    "amount": 10.5,
    "currency": "Bs", // Enviar "Bs" para cobrar el monto exacto en bolívares (o "USD" para conversión BCV)
    "metadata": {
      "user_id": "123",
      "order_id": "456"
    }
}
```

#### Ejemplo de respuesta
```json
{
  "success": true,
  "data": {
    "id": "c00000000000000ny",
    "title": "Pago de prueba",
    "description": "Pago de producto X",
    "amount": 10.5,
    "currency": "Bs",
    "url": "https://venepagos.com.ve/pagar/c00000000000000ny",
    "isActive": true,
    "expiresAt": null,
    "createdAt": "2024-07-23T12:00:00.000Z",
    "metadata": {
      "user_id": "123",
      "order_id": "456"
    }
  },
  "message": "Payment link creado exitosamente"
}
```

### Códigos de respuesta y descripción

| Código | Descripción                                                                                   |
|--------|----------------------------------------------------------------------------------------------|
| 201    | **Creado correctamente.** El enlace de pago fue creado exitosamente y se retorna en la respuesta. |
| 400    | **Solicitud inválida.** Los datos enviados no cumplen con la validación (faltan campos, formato incorrecto, etc). |
| 401    | **No autorizado.** Faltó el header de autorización, el API Key es inválido, expirado o con formato incorrecto. |
| 403    | **Prohibido.** El usuario asociado al API Key está inactivo.                                  |
| 405    | **Método no permitido.** Se intentó acceder al endpoint con un método diferente a POST.        |
| 500    | **Error interno del servidor.** Ocurrió un error inesperado en el backend.                    |

#### Ejemplo de cada respuesta

- **201 Created**
  ```json
  {
    "success": true,
    "data": { ... },
    "message": "Payment link creado exitosamente"
  }
  ```

- **400 Bad Request**
  ```json
  {
    "success": false,
    "error": "Datos de entrada inválidos",
    "details": [
      { "field": "title", "message": "El título debe tener al menos 3 caracteres" }
    ]
  }
  ```

- **401 Unauthorized**
  ```json
  {
    "success": false,
    "error": "Token de autorización requerido. Usar: Bearer <api_key>"
  }
  ```
  o
  ```json
  {
    "success": false,
    "error": "API key no válido"
  }
  ```

- **403 Forbidden**
  ```json
  {
    "success": false,
    "error": "Usuario inactivo"
  }
  ```

- **405 Method Not Allowed**
  ```json
  {
    "success": false,
    "error": "Método no permitido",
    "message": "Este endpoint solo acepta peticiones POST. Consulta la documentación en /docs/PAYMENT_LINKS_API.md"
  }
  ```

- **500 Internal Server Error**
  ```json
  {
    "success": false,
    "error": "Error interno del servidor",
    "details": "Mensaje de error interno"
  }
  ```

---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

## Ejemplos en diferentes lenguajes

<Tabs>
  <TabItem value="curl" label="cURL">

```bash
curl -X POST "https://venepagos.com.ve/api/public/payment-links/create" \
  -H "Authorization: Bearer tu_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Pago de prueba",
    "description": "Pago de producto X",
    "amount": 10.5,
    "currency": "Bs",
    "metadata": {"user_id": "123", "order_id": "456"}
  }'
```

  </TabItem>
  <TabItem value="python" label="Python (requests)">

```python
import requests

url = "https://venepagos.com.ve/api/public/payment-links/create"
headers = {
    "Authorization": "Bearer tu_api_key",
    "Content-Type": "application/json"
}
data = {
    "title": "Pago de prueba",
    "description": "Pago de producto X",
    "amount": 10.5,
    "currency": "Bs",
    "metadata": {"user_id": "123", "order_id": "456"}
}
response = requests.post(url, json=data, headers=headers)
print(response.json())
```

  </TabItem>
  <TabItem value="javascript" label="JavaScript (fetch)">

```js
fetch("https://venepagos.com.ve/api/public/payment-links/create", {
  method: "POST",
  headers: {
    "Authorization": "Bearer tu_api_key",
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    title: "Pago de prueba",
    description: "Pago de producto X",
    amount: 10.5,
    currency: "Bs",
    metadata: { user_id: "123", order_id: "456" }
  })
})
  .then(res => res.json())
  .then(data => console.log(data));
```

  </TabItem>
  <TabItem value="node" label="Node.js (axios)">

```js
const axios = require('axios');

axios.post(
  'https://venepagos.com.ve/api/public/payment-links/create',
  {
    title: 'Pago de prueba',
    description: 'Pago de producto X',
    amount: 10.5,
    currency: 'Bs',
    metadata: { user_id: '123', order_id: '456' }
  },
  {
    headers: {
      Authorization: 'Bearer tu_api_key',
      'Content-Type': 'application/json'
    }
  }
).then(response => {
  console.log(response.data);
});
```

  </TabItem>
  <TabItem value="dart" label="Dart (http)">

```dart
import 'package:http/http.dart' as http;
import 'dart:convert';

void main() async {
  final url = Uri.parse('https://venepagos.com.ve/api/public/payment-links/create');
  final headers = {
    'Authorization': 'Bearer tu_api_key',
    'Content-Type': 'application/json',
  };
  final body = jsonEncode({
    'title': 'Pago de prueba',
    'description': 'Pago de producto X',
    'amount': 10.5,
    'currency': 'Bs',
    'metadata': {'user_id': '123', 'order_id': '456'}
  });

  final response = await http.post(url, headers: headers, body: body);
  print(response.body);
}
```

  </TabItem>
</Tabs>

### Guardar enlace de pago

- Guarda la URL recibida (`data.url`) para luego usarla.


