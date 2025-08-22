---
sidebar_position: 2
---

# Eventos

Los webhooks de Venepagos te permiten recibir notificaciones en tiempo real sobre diferentes eventos que ocurren en tu cuenta. Cada evento tiene un identificador único y contiene datos específicos relevantes al tipo de evento.

#### Eventos de proceso de cobro

| Evento | Identificador | Descripción | Cuándo se envía |
|--------|---------------|-------------|-----------------|
| **Pago creado** | `payment.created` | Se ha iniciado un nuevo proceso de pago | Cuando un usuario accede a un enlace de pago y comienza el proceso |
| **Pago completado** | `payment.completed` | Un pago se ha procesado exitosamente | Cuando el pago se confirma y se acredita en tu cuenta |
| **Pago con reintento** | `payment.retry` | Indica que no debes cerrar el flujo; el usuario puede reintentar | Al cargar la pantalla de confirmación con error; si no hay acción en 5s se enviará `payment.failed` |
| **Pago fallido** | `payment.failed` | Un pago ha fallado por algún motivo | Cuando hay un error en el procesamiento del pago |
| **Pago cancelado** | `payment.cancelled` | Un pago ha sido cancelado por el usuario | Cuando el usuario cancela el proceso de pago |

Notas sobre `payment.retry`:

- Se emite una sola vez al cargar la pantalla de confirmación con error.
- Si el usuario pulsa “Volver a intentar” antes de 5s, no se enviará `payment.failed`.
- Si no hay acción en 5s, se enviará automáticamente `payment.failed`.

#### Eventos de Enlaces de Pago

| Evento | Identificador | Descripción | Cuándo se envía |
|--------|---------------|-------------|-----------------|
| **Enlace creado** | `paymentlink.created` | Se ha creado un nuevo enlace de pago | Cuando creas un enlace de pago desde tu API o portal |
| **Enlace actualizado** | `paymentlink.updated` | Se ha modificado un enlace de pago existente | Cuando actualizas la configuración de un enlace |
| **Enlace eliminado** | `paymentlink.deleted` | Se ha eliminado un enlace de pago | Cuando eliminas un enlace de pago |

#### Cómo funcionan los eventos

1. **Selección de eventos:** Al crear un webhook, puedes seleccionar qué eventos específicos quieres recibir. Solo recibirás notificaciones de los eventos que hayas marcado.

2. **Envío automático:** Cuando ocurre un evento seleccionado, Venepagos envía automáticamente una notificación HTTP POST a tu endpoint configurado.

3. **Reintentos:** Si tu servidor no responde correctamente (HTTP 2xx), el webhook se reintentará hasta 3 veces con intervalos de 1s, 5s y 25s.

4. **Idempotencia:** Es importante que tu endpoint procese los webhooks de forma idempotente, ya que podrías recibir el mismo evento múltiples veces debido a reintentos.
