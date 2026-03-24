# Guía Técnica: API de Notificaciones WhatsApp (Twilio)

Este documento detalla la hoja de ruta para la implementación de un sistema de notificaciones vía WhatsApp para organizaciones sin fines de lucro (Juntas Comunales), garantizando el cumplimiento de las normativas (**Compliance**) y la estabilidad operativa en Venezuela.

---

## 1. Configuración de Cuenta y Proveedores

Para operar legalmente en Venezuela y minimizar fricciones técnicas, seguiremos el flujo oficial a través de **Proveedores de Soluciones de Negocios (BSPs)** como Twilio.

### ¿Por qué usar un Proveedor Verificado (Twilio)?
* **Evita validaciones complejas:** Al usar Twilio, te saltas gran parte de la burocracia directa con Meta/Facebook. El proveedor ya está verificado ante Meta, lo que acelera la aprobación de tu cuenta de WhatsApp Business.
* **Estabilidad en Venezuela:** Gestionan la infraestructura técnica, permitiendo que tu API solo se preocupe por enviar el JSON, sin lidiar con los servidores de WhatsApp.

### Pasos para la Apertura de Cuenta:
1.  **Registro:** Crear cuenta en [Twilio.com](https://www.twilio.com).
2.  **Verificación de Identidad:** Twilio solicitará verificar tu identidad.
3.  **Habilitar el Sandbox:** Antes de comprar un número, se debe usar el *Twilio Sandbox for WhatsApp* para pruebas técnicas.
4.  **Compra de Número (Sender):** Se debe adquirir un número con capacidad SMS/Voz. 

> [!CAUTION]
> ### ⚠️ Advertencia Crítica de la Cuenta (Cloud API)
> Una vez que un número de teléfono se registra y habilita para el **Cloud API de WhatsApp** (a través de Twilio o directamente con Meta), **dicho número queda desactivado en la aplicación móvil de WhatsApp Business**. 
> * No podrás usar el teléfono para chatear manualmente desde la app. 
> * Toda la comunicación deberá gestionarse exclusivamente a través de la **API** o de un **Dashboard** que consuma dicha API.

---

## 2. Gestión de Plantillas (Templates) y Validación

WhatsApp prohíbe el envío de mensajes "libres" para iniciar conversaciones. Es obligatorio el uso de **Plantillas**.

### ¿Qué es una Plantilla?
Es un formato de mensaje pre-aprobado por Meta.
* **Ejemplo de Estructura:** `"Hola {{1}}, la Junta Comunal informa que la próxima asamblea será el día {{2}}. Puedes ver el orden del día aquí: {{3}}"`
* **Validación:** Una vez creada en el panel de Twilio, Meta tarda entre **2 minutos y 24 horas** en aprobarla.

### Reglas para Enlaces (Links):
* Los enlaces deben ser **HTTPS**.
* No usar acortadores como `bit.ly`. Es mejor usar la URL completa o un subdominio propio.
* El enlace debe estar al final del mensaje para mejorar la previsualización.

---

## 3. Arquitectura de la API

El backend (Django REST Framework) será el cerebro que gestione quién recibe qué mensaje y cuándo.

### Endpoint Principal:
`POST /api/v1/notifications/send/`

**Payload sugerido:**
```json
{
    "to": "+584141234567",
    "template_sid": "HXxxxxxxxxxxxx",
    "variables": ["Víctor", "25 de Marzo", "https://junta.com/acta01"]
}
```

---

## 4. Prevención de Bloqueos

Para evitar que la cuenta sea marcada como **SPAM**, aplicaremos las siguientes "Buenas Prácticas":

| Riesgo | Solución / Prevención |
| :--- | :--- |
| **Mensajes Masivos Irrelevantes** | No enviar más de 2 notificaciones por semana a la misma persona para no saturar. |
| **Uso de Enlaces Sospechosos** | Asegurar que el dominio del enlace coincida con el nombre de la organización. |
| **Falta de Interacción** | Si los usuarios nunca responden, WhatsApp baja la "calificación de calidad". Fomenta que respondan con un "Recibido". |