---
sidebar_position: 4
---

# Obtener links de pago

Permite obtener la lista de todos los enlaces de pago creados por el usuario autenticado mediante API Key. Soporta paginación y filtrado por estado.

- **Endpoint:** `/api/public/payment-links/list`
- **Método:** `GET`
- **Headers:**
  - `Authorization: Bearer TU_API_KEY`
- **Parámetros de consulta opcionales:**
  - `limit`: Número máximo de resultados a devolver (por defecto 50, máximo 100)
  - `offset`: Desde qué posición empezar a devolver resultados (por defecto 0)
  - `isActive`: Filtra por estado (`true` o `false`)

#### Ejemplo de cURL
```bash
curl -X GET 'https://venepagos.com.ve/api/public/payment-links/list?limit=10&offset=0&isActive=true' \
  -H 'Authorization: Bearer TU_API_KEY'
```

#### Ejemplo de respuesta exitosa
```json
{
  "success": true,
  "data": {
    "paymentLinks": [
      {
        "id": "c0000000000000000ny",
        "title": "Pago de prueba",
        "description": "Pago de producto X",
        "amount": 10.5,
        "currency": "USD",
        "isActive": true,
        "expiresAt": null,
        "createdAt": "2024-07-23T12:00:00.000Z",
        "updatedAt": "2024-07-23T12:00:00.000Z",
        "url": "https://venepagos.com.ve/pagar/c0000000000000000ny"
      }
      // ...más enlaces
    ],
    "pagination": {
      "total": 25,
      "count": 10,
      "limit": 10,
      "offset": 0,
      "hasMore": true
    }
  }
}
```

#### Códigos de respuesta y descripción

| Código | Descripción                                                                                   |
|--------|----------------------------------------------------------------------------------------------|
| 200    | **OK.** Lista de enlaces de pago devuelta correctamente.                                      |
| 401    | **No autorizado.** API Key inválido, expirado o no enviado.                                   |
| 403    | **Prohibido.** El usuario asociado al API Key está inactivo.                                  |
| 500    | **Error interno del servidor.** Ocurrió un error inesperado en el backend.                    |

#### Ejemplo de error (API Key inválido)
```json
{
  "success": false,
  "error": "API key no válido"
}
```

#### Ejemplo de error (usuario inactivo)
```json
{
  "success": false,
  "error": "Usuario inactivo"
}
```

#### Ejemplo de error (error interno)
```json
{
  "success": false,
  "error": "Error interno del servidor",
  "details": "Mensaje de error interno"
}
```

