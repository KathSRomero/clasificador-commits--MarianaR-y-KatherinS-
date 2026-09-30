Arquitectura y seguridad

# Manual Técnico - Clasificador de Commits

## 1. Arquitectura
La solución está compuesta por las siguientes piezas:
* **Cliente:** Navegador web (o Postman) desde donde el usuario envía el texto.
* **API (FastAPI - Puerto 8000):** Es el servidor principal que recibe la petición y decide qué hacer.
* **Motores de Clasificación:**
  * **Motor eco:** Usa reglas de texto simples (no consume memoria).
  * **Motor Ollama (Puerto 11434):** Usa el modelo de inteligencia artificial local.
* **Base de Datos (PostgreSQL - Puerto 5432):** Contenedor de Docker que guarda el historial de todas las clasificaciones realizadas.

## 2. Seguridad
* **Puertos expuestos:** Se expone el 8000 para que los usuarios usen la API, el 5432 para la conexión a la base de datos, y el 11434 interno para consultar a Ollama.
* **Roles en Base de Datos:** Por seguridad, la API no se conecta como administrador. Usa el rol `app_ia` que tiene privilegios mínimos: solo puede consultar (`SELECT`) e insertar datos (`INSERT`). No tiene permisos para borrar o dañar la tabla.
* **Manejo de secretos:** Las contraseñas de la base de datos viven en el archivo oculto `.env`. Este archivo jamás se sube a GitHub porque está bloqueado mediante el archivo `.gitignore`.
* **Plan de acción ante filtraciones:** Si una contraseña llega a filtrarse, el protocolo es detener la aplicación, cambiar las claves en el archivo local `.env` y volver a levantar el contenedor.
