Parte de un proyecto del proyecto final de la carrera de automatizacion IA de coderhouse.
En este caso se eligio un consultor IA para asesoramiento de adaptacion de cvs y afinidad a ofertas laborales.

# Checkpoint 4 — Integraciones Avanzadas e Interconexión de Sistemas


Este checkpoint amplía el sistema multi-agente desarrollado en los módulos anteriores incorporando integraciones con herramientas externas reales y controles preventivos para garantizar una comunicación segura y controlada entre el sistema agéntico y los servicios externos.

El workflow fue desarrollado sobre **n8n**, manteniendo la arquitectura del proyecto integrador y agregando integraciones con **Gmail, HubSpot y Slack**.

---

## Objetivos

* Integrar el sistema multi-agente con herramientas externas reales.
* Implementar autenticación mediante OAuth2.
* Aplicar el principio de mínimo privilegio en las operaciones.
* Evitar respuestas automáticas en cadena y bucles infinitos.
* Evitar la creación de contactos duplicados en el CRM.
* Incorporar una barrera de seguridad mediante revisión humana.
* Limpiar y reducir los datos enviados a los canales de comunicación.
* Validar mediante pruebas manuales la estabilidad del workflow.

---

## Arquitectura de integraciones

El workflow agrega tres servicios externos principales:

| Herramienta | Función                                         |
| ----------- | ----------------------------------------------- |
| **Gmail**   | Entrada de consultas y generación de borradores |
| **HubSpot** | Gestión y persistencia de contactos             |
| **Slack**   | Notificación interna al equipo de operaciones   |

Estas herramientas se integran con el sistema de agentes y la memoria persistente utilizada por el proyecto.

---

## Gmail

Gmail funciona como casilla de entrada y salida controlada del sistema.

### Entrada

El workflow comienza mediante un **Gmail Trigger**, que recibe los mensajes enviados a la casilla configurada.

Inmediatamente después se incorpora un nodo **IF** que permite filtrar mensajes que no deberían ingresar al procesamiento automático.

Se contemplan casos como:

* `Auto-reply`
* `Out of office`
* `Undeliverable`
* direcciones `no-reply@`

Esto permite evitar ciclos de auto-respuesta y procesamientos innecesarios.

### Salida

La respuesta generada por el sistema no se envía directamente al usuario.

Se utiliza exclusivamente la operación:

**Gmail → Create Draft**

De esta manera, el resultado queda almacenado como borrador y requiere intervención humana antes de su emisión definitiva.

Esto implementa el principio de **Human-in-the-Loop**.

---

## HubSpot

HubSpot se utiliza como CRM para almacenar y consultar la información de los contactos.

Antes de realizar una operación de creación se incorpora una búsqueda previa:

**Search contact → IF → Create contact**

El sistema verifica primero si el contacto ya existe.

### Contacto existente

Si el contacto es encontrado, el workflow evita crear un nuevo registro.

### Contacto inexistente

Si no existe, se ejecuta la operación de creación.

Este mecanismo funciona como una compuerta preventiva contra duplicados y permite evitar errores asociados a registros repetidos, incluyendo el conflicto HTTP 409.

De esta forma, HubSpot funciona como una fuente de verdad para la información de contacto.

---

## Slack

Slack funciona como canal interno de notificación para el equipo de operaciones.

Una vez procesada la consulta, el workflow prepara los datos necesarios y los envía al canal configurado.

El mensaje puede incluir información como:

* Canal de origen.
* Dirección de correo.
* Tipo de consulta.
* Pregunta recibida.
* Respuesta generada por el sistema.

Antes del envío se utilizan nodos de **Edit Fields** para estructurar el payload y limitar la información transmitida al canal a los campos necesarios.

Esto evita enviar innecesariamente objetos o información adicional que el canal no necesita procesar.

---

## Memoria y contexto

El workflow mantiene la arquitectura de memoria desarrollada en los módulos anteriores.

La información procesada se consulta y actualiza mediante Airtable, permitiendo conservar el contexto de las interacciones y utilizarlo durante las ejecuciones posteriores.

La actualización de memoria se realiza en diferentes etapas del procesamiento para mantener sincronizados los datos relevantes de la conversación.

---

## Inteligencia artificial

El procesamiento de las consultas continúa utilizando los agentes y modelos configurados en los módulos anteriores.

La información obtenida desde las herramientas externas se estructura antes de ser enviada al procesamiento de IA.

Posteriormente, la respuesta generada es formateada antes de:

1. Actualizar la memoria.
2. Generar el borrador en Gmail.
3. Notificar al equipo mediante Slack.

---

## Controles de seguridad y gobernanza

El workflow incorpora diferentes controles preventivos:

### 1. Filtro de correos automáticos

El nodo IF posterior al Gmail Trigger evita procesar determinados mensajes automáticos.

### 2. Prevención de duplicados

La búsqueda previa en HubSpot evita crear contactos que ya existen.

### 3. Human-in-the-Loop

Las respuestas no se envían automáticamente. Se generan como borradores mediante Gmail.

### 4. Minimización del payload

Los nodos Edit Fields permiten seleccionar únicamente los datos necesarios antes de enviarlos a los servicios externos.

### 5. Mínimo privilegio

Las operaciones de cada integración se limitan a las funciones necesarias para el flujo operativo.

---

## Pruebas realizadas

Se realizaron pruebas individuales y de integración sobre los principales componentes.

### Gmail

* Recepción de mensajes mediante Gmail Trigger.
* Evaluación mediante IF.
* Generación de borradores mediante Create Draft.

### HubSpot

* Búsqueda de contactos existentes.
* Verificación del comportamiento cuando el contacto ya existe.
* Creación de contactos cuando no existe.
* Verificación de prevención de duplicados.

### Slack

* Autenticación de la cuenta.
* Selección del canal.
* Ejecución individual del nodo.
* Recepción correcta del mensaje en el canal configurado.

### Workflow

Se realizaron varias ejecuciónes de prueba sobre el flujo completo, verificando el correcto encadenamiento de los nodos incluido y la ejecución satisfactoria de las etapas principales.


## Relación con el Proyecto Final

Este checkpoint continúa la evolución del mismo sistema desarrollado durante los módulos anteriores.

La arquitectura incorpora progresivamente:

**Entrada → Orquestación → Agentes → Memoria → Herramientas externas → Controles → Validación humana → Salidas**


---

## Herramientas utilizadas hasta ahora:

* n8n
* Gmail
* HubSpot
* Slack
* Airtable
* Groq / modelos de IA


