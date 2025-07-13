---
sidebar_position: 2
---

# Proceso de cobro

Una vez que tengas la URL del enlace de pago (`data.url`), puedes redirigir al usuario de varias formas según tu plataforma:

- Abrir enlace en una ventana nueva del navegador.
- Crear un componente de WebView y abrir el enlace.
- Abre una ventana emergente del navegador dentro de tu app.

### Formas de procesar cobro en JavaScript:

Supongamos que ya realizaste la solicitud a la API y recibiste la respuesta con la URL del enlace de pago en `data.url`:

```js
// Ejemplo de cómo obtener la URL del enlace de pago desde la respuesta de la API
const response = await fetch('https://venepagos.com.ve/api/public/payment-links/create', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer tu_api_key',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    title: 'Pago de prueba',
    description: 'Pago de producto X',
    amount: 10.5,
    currency: 'USD',
    metadata: { user_id: '123', order_id: '456' }
  })
});
const data = await response.json();
const paymentUrl = data.data.url;
```

Ahora puedes usar `paymentUrl` en cualquiera de los siguientes métodos:

#### 1. Abrir en ventana nueva.

```jsx
// Abre el enlace de pago en una nueva pestaña o ventana
window.open(paymentUrl, '_blank');
```

#### 2. Crear un componente y abrir un WebView.

```jsx
// Usando un iframe para simular un WebView en web
function PaymentWebView({ paymentUrl }) {
  return (
    <iframe
      src={paymentUrl}
      style={{ width: '100%', height: '600px', border: 'none' }}
      title="Pago Venepagos"
    />
  );
}

// Uso:
// <PaymentWebView paymentUrl={paymentUrl} />
```

#### 3. Abrir una ventana emergente del navegador dentro de la app.

```jsx
// Puedes controlar el tamaño y características de la ventana emergente
const popup = window.open(
  paymentUrl,
  'PagoVenepagos',
  'width=500,height=700,scrollbars=yes,resizable=yes'
);
```


## Webhooks: Recibe eventos de pago

Cuando el usuario paga (o el pago falla/cancela), Venepagos enviará un evento a tu endpoint de webhook configurado.

#### Ejemplo de payload de webhook (evento `payment.completed`)
```json
{
  "event": "payment.completed",
  "data": {
    "id": "txn_1752286810657_m40p4d7c0dd",
    "amount": 10.5,
    "currency": "USD",
    "status": "completed",
    "paymentLinkId": "cmczm8edj0005c97dur7412ny",
    "customerEmail": "cliente@email.com",
    "customerName": "Juan Pérez",
    "reference": "REF123456",
    "timestamp": "2024-07-23T12:05:00.000Z"
  },
  "webhook_id": "wh_abc123",
  "timestamp": "2024-07-23T12:05:00.000Z"
}
```

#### Otros eventos posibles
- `payment.failed`
- `payment.cancelled`

Consulta la sección [Webhooks](#webhooks) para más detalles sobre la validación y manejo de estos eventos.

---

> **Resumen:**
> 1. Crea el enlace de pago → 2. Redirige al usuario → 3. Recibe el evento por webhook

---