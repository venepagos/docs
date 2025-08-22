---
sidebar_position: 3
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Ejemplos de uso


### Ejemplos de implementación del endpoint webhook

Aquí tienes ejemplos de cómo implementar el endpoint `/webhook` en diferentes lenguajes para recibir y procesar las notificaciones de Venepagos:

#### Node.js (Express)

```javascript
const express = require('express');
const crypto = require('crypto');
const app = express();

app.use(express.json());

// Tu secret del webhook (configurado en Venepagos)
const WEBHOOK_SECRET = 'tu_webhook_secret_aqui';

// Función para validar la firma del webhook
function validateWebhookSignature(payload, signature, secret) {
  const expectedSignature = 'sha256=' + crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}

app.post('/webhook', (req, res) => {
  try {
    const payload = JSON.stringify(req.body);
    const signature = req.headers['x-webhook-signature'];
    const event = req.headers['x-webhook-event'];
    
    // Validar la firma si tienes un secret configurado
    if (WEBHOOK_SECRET && signature) {
      if (!validateWebhookSignature(payload, signature, WEBHOOK_SECRET)) {
        console.error('Firma de webhook inválida');
        return res.status(401).json({ error: 'Firma inválida' });
      }
    }
    
    // Procesar el evento según su tipo
    switch (event) {
      case 'payment.completed':
        console.log('Pago completado:', req.body.data);
        // Aquí procesas el pago completado
        // Ejemplo: actualizar base de datos, enviar email, etc.
        break;
      
      case 'payment.retry':
        console.log('Pago en reintento:', req.body.data);
        // Mantén el flujo abierto; no marques como fallo definitivo aún
        break;
        
      case 'payment.failed':
        console.log('Pago fallido:', req.body.data);
        // Procesar pago fallido
        break;
        
      case 'paymentlink.created':
        console.log('Enlace creado:', req.body.data);
        // Procesar nuevo enlace de pago
        break;
        
      default:
        console.log('Evento no manejado:', event);
    }
    
    // Responder con éxito
    res.status(200).json({ received: true });
    
  } catch (error) {
    console.error('Error procesando webhook:', error);
    res.status(500).json({ error: 'Error interno' });
  }
});

app.listen(3000, () => {
  console.log('Servidor webhook corriendo en puerto 3000');
});
```

#### Python (Flask)

```python
from flask import Flask, request, jsonify
import hmac
import hashlib
import json

app = Flask(__name__)

# Tu secret del webhook (configurado en Venepagos)
WEBHOOK_SECRET = 'tu_webhook_secret_aqui'

def validate_webhook_signature(payload, signature, secret):
    """Valida la firma del webhook"""
    expected_signature = 'sha256=' + hmac.new(
        secret.encode('utf-8'),
        payload.encode('utf-8'),
        hashlib.sha256
    ).hexdigest()
    
    return hmac.compare_digest(signature, expected_signature)

@app.route('/webhook', methods=['POST'])
def webhook():
    try:
        payload = json.dumps(request.json, separators=(',', ':'))
        signature = request.headers.get('X-Webhook-Signature')
        event = request.headers.get('X-Webhook-Event')
        
        # Validar la firma si tienes un secret configurado
        if WEBHOOK_SECRET and signature:
            if not validate_webhook_signature(payload, signature, WEBHOOK_SECRET):
                print('Firma de webhook inválida')
                return jsonify({'error': 'Firma inválida'}), 401
        
        data = request.json.get('data', {})
        
        # Procesar el evento según su tipo
        if event == 'payment.completed':
            print(f'Pago completado: {data}')
            # Aquí procesas el pago completado
            # Ejemplo: actualizar base de datos, enviar email, etc.
            
        elif event == 'payment.retry':
            print(f'Pago en reintento: {data}')
            # Mantén el flujo abierto; no marques como fallo definitivo aún
            
        elif event == 'payment.failed':
            print(f'Pago fallido: {data}')
            # Procesar pago fallido
            
        elif event == 'paymentlink.created':
            print(f'Enlace creado: {data}')
            # Procesar nuevo enlace de pago
            
        else:
            print(f'Evento no manejado: {event}')
        
        # Responder con éxito
        return jsonify({'received': True}), 200
        
    except Exception as e:
        print(f'Error procesando webhook: {e}')
        return jsonify({'error': 'Error interno'}), 500

if __name__ == '__main__':
    app.run(debug=True, port=3000)
```

#### PHP

```php
<?php
// Tu secret del webhook (configurado en Venepagos)
$WEBHOOK_SECRET = 'tu_webhook_secret_aqui';

// Función para validar la firma del webhook
function validateWebhookSignature($payload, $signature, $secret) {
    $expectedSignature = 'sha256=' . hash_hmac('sha256', $payload, $secret);
    return hash_equals($signature, $expectedSignature);
}

// Obtener el payload y headers
$payload = file_get_contents('php://input');
$signature = $_SERVER['HTTP_X_WEBHOOK_SIGNATURE'] ?? '';
$event = $_SERVER['HTTP_X_WEBHOOK_EVENT'] ?? '';

// Validar la firma si tienes un secret configurado
if ($WEBHOOK_SECRET && $signature) {
    if (!validateWebhookSignature($payload, $signature, $WEBHOOK_SECRET)) {
        http_response_code(401);
        echo json_encode(['error' => 'Firma inválida']);
        exit;
    }
}

// Decodificar el payload
$data = json_decode($payload, true);
$eventData = $data['data'] ?? [];

// Procesar el evento según su tipo
switch ($event) {
    case 'payment.completed':
        error_log("Pago completado: " . json_encode($eventData));
        // Aquí procesas el pago completado
        // Ejemplo: actualizar base de datos, enviar email, etc.
        break;
    
    case 'payment.retry':
        error_log("Pago en reintento: " . json_encode($eventData));
        // Mantén el flujo abierto; no marques como fallo definitivo aún
        break;
        
    case 'payment.failed':
        error_log("Pago fallido: " . json_encode($eventData));
        // Procesar pago fallido
        break;
        
    case 'paymentlink.created':
        error_log("Enlace creado: " . json_encode($eventData));
        // Procesar nuevo enlace de pago
        break;
        
    default:
        error_log("Evento no manejado: " . $event);
}

// Responder con éxito
http_response_code(200);
echo json_encode(['received' => true]);
?>
```

#### Java (Spring Boot)

```java
import org.springframework.web.bind.annotation.*;
import org.springframework.http.*;
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.util.Base64;

@RestController
public class WebhookController {
    
    // Tu secret del webhook (configurado en Venepagos)
    private static final String WEBHOOK_SECRET = "tu_webhook_secret_aqui";
    
    @PostMapping("/webhook")
    public ResponseEntity<Map<String, Object>> webhook(
            @RequestBody String payload,
            @RequestHeader("X-Webhook-Signature") String signature,
            @RequestHeader("X-Webhook-Event") String event) {
        
        try {
            // Validar la firma si tienes un secret configurado
            if (WEBHOOK_SECRET != null && !WEBHOOK_SECRET.isEmpty() && signature != null) {
                if (!validateWebhookSignature(payload, signature, WEBHOOK_SECRET)) {
                    return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                        .body(Map.of("error", "Firma inválida"));
                }
            }
            
            // Procesar el evento según su tipo
            switch (event) {
                case "payment.completed":
                    System.out.println("Pago completado: " + payload);
                    // Aquí procesas el pago completado
                    // Ejemplo: actualizar base de datos, enviar email, etc.
                    break;
                
                case "payment.retry":
                    System.out.println("Pago en reintento: " + payload);
                    // Mantén el flujo abierto; no marques como fallo definitivo aún
                    break;
                    
                case "payment.failed":
                    System.out.println("Pago fallido: " + payload);
                    // Procesar pago fallido
                    break;
                    
                case "paymentlink.created":
                    System.out.println("Enlace creado: " + payload);
                    // Procesar nuevo enlace de pago
                    break;
                    
                default:
                    System.out.println("Evento no manejado: " + event);
            }
            
            // Responder con éxito
            return ResponseEntity.ok(Map.of("received", true));
            
        } catch (Exception e) {
            System.err.println("Error procesando webhook: " + e.getMessage());
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(Map.of("error", "Error interno"));
        }
    }
    
    private boolean validateWebhookSignature(String payload, String signature, String secret) {
        try {
            Mac mac = Mac.getInstance("HmacSHA256");
            SecretKeySpec secretKey = new SecretKeySpec(secret.getBytes(), "HmacSHA256");
            mac.init(secretKey);
            
            String expectedSignature = "sha256=" + 
                Base64.getEncoder().encodeToString(mac.doFinal(payload.getBytes()));
            
            return signature.equals(expectedSignature);
        } catch (NoSuchAlgorithmException | InvalidKeyException e) {
            return false;
        }
    }
}
```

#### Consideraciones importantes

1. **Validación de firma:** Siempre valida la firma del webhook para asegurar que proviene de Venepagos.
2. **Idempotencia:** Procesa los webhooks de forma que no cause problemas si recibes el mismo evento múltiples veces.
3. **Respuesta rápida:** Responde con HTTP 200-299 en menos de 5 segundos.
4. **Logging:** Registra todos los eventos para debugging y auditoría.
5. **Manejo de errores:** Maneja las excepciones apropiadamente para evitar que el webhook falle.

---

## Eventos de Webhook y payloads

<Tabs>
  <TabItem value="payment.created" label="payment.created">
    <strong>payment.created</strong><br/>
    Se dispara cuando se crea un nuevo intento de pago asociado a un enlace de pago.
    <details>
      <summary>Ejemplo de payload</summary>

```json
{
  "event": "payment.created",
  "data": {
    "id": "txn_1752286810657_m40p4d7c0dd",
    "amount": 100.50,
    "currency": "USD",
    "status": "created",
    "paymentLinkId": "cmczm8edj0005c97dur7412ny",
    "customerEmail": "cliente@email.com",
    "customerName": "Juan Pérez",
    "timestamp": "2024-01-15T10:30:00.000Z"
  },
  "webhook_id": "wh_abc123",
  "timestamp": "2024-01-15T10:30:00.000Z"
}
```
    </details>
  </TabItem>
  <TabItem value="payment.completed" label="payment.completed">
    <strong>payment.completed</strong><br/>
    Se dispara cuando un pago ha sido completado exitosamente a través de un enlace de pago.
    <details>
      <summary>Ejemplo de payload</summary>

```json
{
  "event": "payment.completed",
  "data": {
    "id": "txn_1752286810657_m40p4d7c0dd",
    "amount": 100.50,
    "currency": "USD",
    "status": "completed",
    "paymentLinkId": "cmczm8edj0005c97dur7412ny",
    "customerEmail": "cliente@email.com",
    "customerName": "Juan Pérez",
    "reference": "REF123456",
    "timestamp": "2024-01-15T10:35:00.000Z"
  },
  "webhook_id": "wh_abc123",
  "timestamp": "2024-01-15T10:35:00.000Z"
}
```
    </details>
  </TabItem>
  <TabItem value="payment.failed" label="payment.failed">
    <strong>payment.failed</strong><br/>
    Se dispara cuando un intento de pago falla.
    <details>
      <summary>Ejemplo de payload</summary>

```json
{
  "event": "payment.failed",
  "data": {
    "id": "txn_1752286810657_m40p4d7c0dd",
    "amount": 100.50,
    "currency": "USD",
    "status": "failed",
    "paymentLinkId": "cmczm8edj0005c97dur7412ny",
    "customerEmail": "cliente@email.com",
    "customerName": "Juan Pérez",
    "failureReason": "Saldo insuficiente en la cuenta",
    "timestamp": "2024-01-15T10:32:00.000Z"
  },
  "webhook_id": "wh_abc123",
  "timestamp": "2024-01-15T10:32:00.000Z"
}
```
    </details>
  </TabItem>
  <TabItem value="payment.cancelled" label="payment.cancelled">
    <strong>payment.cancelled</strong><br/>
    Se dispara cuando el usuario cancela el proceso de pago antes de completarlo.
    <details>
      <summary>Ejemplo de payload</summary>

```json
{
  "event": "payment.cancelled",
  "data": {
    "id": "txn_1752286810657_m40p4d7c0dd",
    "amount": 100.50,
    "currency": "USD",
    "status": "cancelled",
    "paymentLinkId": "cmczm8edj0005c97dur7412ny",
    "customerEmail": "cliente@email.com",
    "customerName": "Juan Pérez",
    "timestamp": "2024-01-15T10:33:00.000Z"
  },
  "webhook_id": "wh_abc123",
  "timestamp": "2024-01-15T10:33:00.000Z"
}
```
    </details>
  </TabItem>
  <TabItem value="paymentlink.created" label="paymentlink.created">
    <strong>paymentlink.created</strong><br/>
    Se dispara cuando se crea un nuevo enlace de pago.
    <details>
      <summary>Ejemplo de payload</summary>

```json
{
  "event": "paymentlink.created",
  "data": {
    "id": "cmczm8edj0005c97dur7412ny",
    "title": "Pago de Producto X",
    "description": "Pago para el producto con ID 123",
    "amount": 100.50,
    "currency": "USD",
    "url": "https://venepagos.com.ve/pagar/cmczm8edj0005c97dur7412ny",
    "isActive": true,
    "expiresAt": "2024-01-22T10:30:00.000Z",
    "metadata": {
      "user_id": "123",
      "order_id": "456",
      "product_name": "Producto X"
    },
    "timestamp": "2024-01-15T10:30:00.000Z"
  },
  "webhook_id": "wh_abc123",
  "timestamp": "2024-01-15T10:30:00.000Z"
}
```
    </details>
  </TabItem>
  <TabItem value="paymentlink.updated" label="paymentlink.updated">
    <strong>paymentlink.updated</strong><br/>
    Se dispara cuando se actualiza un enlace de pago existente.
    <details>
      <summary>Ejemplo de payload</summary>

```json
{
  "event": "paymentlink.updated",
  "data": {
    "id": "cmczm8edj0005c97dur7412ny",
    "title": "Pago de Producto X - Actualizado",
    "description": "Pago actualizado para el producto con ID 123",
    "amount": 150.00,
    "currency": "USD",
    "url": "https://venepagos.com.ve/pagar/cmczm8edj0005c97dur7412ny",
    "isActive": true,
    "expiresAt": "2024-01-22T10:30:00.000Z",
    "metadata": {
      "user_id": "123",
      "order_id": "456",
      "product_name": "Producto X",
      "updated_at": "2024-01-15T10:30:00.000Z"
    },
    "timestamp": "2024-01-15T10:30:00.000Z"
  },
  "webhook_id": "wh_abc123",
  "timestamp": "2024-01-15T10:30:00.000Z"
}
```
    </details>
  </TabItem>
  <TabItem value="paymentlink.deleted" label="paymentlink.deleted">
    <strong>paymentlink.deleted</strong><br/>
    Se dispara cuando se elimina un enlace de pago.
    <details>
      <summary>Ejemplo de payload</summary>

```json
{
  "event": "paymentlink.deleted",
  "data": {
    "id": "cmczm8edj0005c97dur7412ny",
    "title": "Pago de Producto X",
    "description": null,
    "amount": null,
    "currency": "USD",
    "url": null,
    "isActive": false,
    "expiresAt": null,
    "metadata": null,
    "timestamp": "2024-01-15T10:30:00.000Z"
  },
  "webhook_id": "wh_abc123",
  "timestamp": "2024-01-15T10:30:00.000Z"
}
```
    </details>
  </TabItem>
</Tabs>