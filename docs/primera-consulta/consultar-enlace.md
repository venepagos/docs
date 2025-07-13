---
sidebar_position: 3
---

# Consultar link de pago

Permite consultar y obtener toda la información asociada a un enlace de pago a partir de su identificador único (ID). Esta funcionalidad es útil para mostrar detalles del cobro, como el monto, la moneda, la descripción, el estado del enlace y cualquier dato adicional que haya sido incluido al momento de su creación. Es ideal para integraciones donde se necesita validar o mostrar información del enlace antes de redirigir al usuario o procesar el pago, permitiendo así una experiencia más transparente y segura para el usuario final.

- **Endpoint:** `/api/public/payment-links/{id}`
- **Método:** `GET`
- **Parámetro:** `id` (en la URL)

#### Ejemplo de cURL
```bash
curl -X GET "https://venepagos.com.ve/api/public/payment-links/cmczm8edj0005c97dur7412ny"
```

#### Ejemplo de respuesta exitosa
```json
{
  "id": "cmczm8edj0005c97dur7412ny",
  "title": "Pago de prueba",
  "description": "Pago de producto X",
  "amount": 10.5,
  "currency": "USD",
  "isActive": true,
  "expiresAt": null,
  "createdAt": "2024-07-23T12:00:00.000Z",
  "userId": "user_abc123",
  "user": {
    "razonSocial": "Mi Empresa C.A.",
    "logo": null
  },
  "metadata": {
    "user_id": "123",
    "order_id": "456"
  }
}
```

#### Códigos de respuesta y descripción

| Código | Descripción                                                                                   |
|--------|----------------------------------------------------------------------------------------------|
| 200    | **OK.** El enlace de pago fue encontrado y se retorna la información.                        |
| 400    | **Solicitud inválida.** El enlace está inactivo, expirado o el ID no fue especificado.       |
| 404    | **No encontrado.** No existe un enlace de pago con ese ID.                                   |
| 500    | **Error interno del servidor.** Ocurrió un error inesperado en el backend.                   |

#### Ejemplo de error (enlace no encontrado)
```json
{
  "error": "Enlace de pago no encontrado"
}
```

#### Ejemplo de error (enlace inactivo o expirado)
```json
{
  "error": "Este enlace de pago no está activo"
}
```

#### Ejemplo de error (ID no especificado)
```json
{
  "error": "ID de enlace de pago no especificado"
}
```


