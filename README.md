# 🤖 HR Buddy - Inmersión Agentes de IA | G10 LAD (Alura Latam)

Este repositorio contiene la solución técnica y la documentación arquitectónica del flujo automatizado en **n8n** desarrollado para el desafío final de la **Inmersión Agentes de IA | G10 LAD**. El proyecto consiste en un asistente virtual de Recursos Humanos (HR Buddy) para la empresa ficticia **ChocolaTech**, integrado nativamente con **Telegram** y respaldado por una base de datos relacional **MySQL**.

El flujo ha sido diseñado bajo principios de ingeniería de software modernos, priorizando la **optimización en el consumo de tokens** y un esquema de **seguridad perimetral (Zero-Trust)** para proteger la confidencialidad de los datos del personal.

---

## 🚀 Acceso al Bot
El asistente se encuentra desplegado y operativo en producción a través del siguiente enlace de Telegram:
👉 [t.me/a_chocolatech_bot](https://t.me/a_chocolatech_bot)

---

## 🏗️ Arquitectura de la Solución

La solución se divide en dos capas fundamentales que resuelven los desafíos de negocio e infraestructura planteados en la inmersión:

### 🛡️ Desafío 1: Filtro y Clasificación de Intenciones perimetral
Para evitar que el **AI Agent** principal procese de forma redundante interacciones fuera de alcance (saludos genéricos, spam, o consultas no autorizadas), se implementó un nodo **Basic LLM Chain** propulsado por el modelo **Cohere** que actúa como un "portero" inteligente de peticiones.

* **Flujo Lógico:**
    1. El `Telegram Trigger` recibe el webhook del usuario.
    2. El mensaje es evaluado por el `Basic LLM Chain` (Clasificador).
    3. Un nodo **If (Condicional)** valida estrictamente el string de salida del modelo.
    4. **Ruta True:** Si la consulta pertenece a Recursos Humanos o es una presentación de identidad, el flujo avanza al `AI Agent`.
    5. **Ruta False:** Si la consulta es ajena, se intercepta y se envía una respuesta estática predefinida, reduciendo a cero el gasto de tokens del agente complejo.

* **Prompt del Clasificador (Basic LLM Chain):**
    ```text
    Analiza el siguiente mensaje del usuario y determina si es una consulta relacionada con Recursos Humanos (vacaciones, políticas, banco de horas, sueldos, normas). ATENCIÓN: Si el usuario simplemente se está presentando o dando su nombre, también considéralo válido.
    Mensaje: {{ $json.message.text }}
    Responde ÚNICAMENTE con la palabra TRUE si cumple con lo anterior. Responde ÚNICAMENTE con la palabra FALSE si es un saludo genérico o un tema ajeno a la empresa.
    ```

### 🔒 Desafío 2: Capa de Seguridad de Acceso de Datos (Zero-Trust)
El segundo requerimiento exigía asegurar que un usuario únicamente pudiese consultar sus propios registros de vacaciones y banco de horas. En lugar de delegar el control de acceso a la capa lógica de la Inteligencia Artificial (lo cual introduce riesgos de *Prompt Injection*), la validación de identidad se delegó directamente al motor de la **Base de Datos MySQL**.

* **Estrategia de Mitigación:**
    Asumiendo un mapeo previo de identidades dentro de la organización, la herramienta del Agente `Select rows from a table in MySQL` fue modificada estructuralmente. La consulta restringe la búsqueda forzando a que el parámetro `chat_id` guardado de forma segura en la tabla coincida exactamente con el identificador dinámico de Telegram del remitente (`message.from.id`).
    
    Si un usuario intenta suplantar otra identidad o consultar datos ajenos, la consulta MySQL devuelve un set de datos vacío (`null`). Al no existir registros vinculados a ese ID de chat, el Agente intercepta la ausencia de datos y dispara una negativa de acceso inequívoca.

* **Prompt Optimizado del Agente de RR. HH. (System Message):**
    ```text
    Eres HR Buddy, asistente virtual de RR. HH. de ChocolaTech. 
    
    REGLAS GLOBALES:
    - Responde SIEMPRE en español y SOLO sobre recursos humanos. 
    - Usa la herramienta Vector Store EXCLUSIVAMENTE para responder dudas sobre políticas generales. No inventes datos.
    
    FLUJO DE ATENCIÓN:
    1. IDENTIFICACIÓN: Si el usuario no indica su nombre completo, pídeselo amablemente.
    2. BÚSQUEDA: Cuando tengas el nombre, usa la herramienta MySQL para buscar sus datos.
    3. RESULTADOS:
       - Si la herramienta devuelve información: Entrégale sus datos de vacaciones/horas.
       - Si la herramienta NO devuelve información (vacío): Esto significa que el empleado no existe o que por motivos de seguridad el usuario no tiene permisos para ver esa información. Responde EXACTAMENTE: "Lo siento, no puedo encontrar tus datos o, por políticas de seguridad, no tienes autorización para ver la información solicitada."
    ```

---

## 🛠️ Stack Tecnológico Utilizado
* **Orquestación de Workflows:** n8n Cloud (Instancia pública para garantizar la conectividad de Webhooks nativos).
* **Modelos de Lenguaje (LLM):** Cohere Chat Model (Framework ReAct para llamadas de herramientas) & Cohere Embeddings.
* **Base de Datos Relacional:** MySQL / Supabase (Estructura de datos optimizada e indexada).
* **Base de Conocimientos:** Simple Vector Store (Políticas internas del corporativo extraídas desde repositorios de configuración).
* **Interfaz de Usuario:** API de Telegram (Bot Dedicado).

---

## 🚀 Instrucciones de Despliegue e Importación

Para replicar este entorno de forma local o en una instancia cloud de n8n, sigue estos pasos:

1.  **Clonar este repositorio:**
    ```bash
    git clone https://github.com/TU_USUARIO/TU_REPO.git
    ```
2.  **Importar el Workflow:**
    * Crea un nuevo flujo en blanco en tu interfaz de **n8n**.
    * Haz clic en el menú de la esquina superior derecha y selecciona **Import from File**.
    * Selecciona el archivo `flujo.json` incluido en la raíz de este proyecto.
3.  **Configurar Credenciales:**
    * Actualiza el token de acceso de Telegram provisto por *BotFather*.
    * Configura la cadena de conexión de tu base de datos MySQL en las herramientas correspondientes.
4.  **Ejecutar:** Activa el flujo para comenzar a escuchar eventos en tiempo real a través del Webhook expuesto de forma segura.

---
*Proyecto desarrollado con fines de certificación académica para Alura Latam - 2026.*
