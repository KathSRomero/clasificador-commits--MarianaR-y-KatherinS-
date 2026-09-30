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

## Respaldo y restauración

### Procedimiento de respaldo

1. Crear la carpeta `backups/` si no existe: `mkdir -p backups`
2. Ejecutar el respaldo:
   ```bash
   docker compose exec -T db pg_dump -U postgres iadb > backups/respaldo_$(date +%F).sql

## Decisiones de diseño y limitaciones

### Decisiones de diseño

- **Dockerfile multi-etapa**: reduce el tamaño final de la imagen. La etapa de construcción (con compiladores) no se incluye en la imagen final.
- **Usuario sin privilegios (`appuser`) en el contenedor**: si la API fuera comprometida, el atacante no tendría permisos de root dentro del contenedor.
- **Motor eco por defecto**: permite que el sistema funcione en cualquier equipo, incluso sin modelo de lenguaje, y sirve de línea base para comparar con el motor ollama.
- **Privilegios mínimos en la base de datos**: el rol `app_ia` solo puede hacer SELECT e INSERT. No puede modificar la estructura ni borrar datos.
- **Variables de entorno**: las credenciales nunca están en el código; se leen desde `.env`.
- **`OLLAMA_KEEP_ALIVE=0`**: obliga al modelo a descargarse de memoria después de cada respuesta, permitiendo que el sistema funcione en equipos con 4 GB de RAM (a costa de mayor latencia).

### Limitaciones conocidas

- **Latencia alta del motor ollama en equipos modestos**: con `OLLAMA_KEEP_ALIVE=0`, cada inferencia tarda entre 2 y 5 segundos.
- **Modelos pequeños tienen precisión limitada**: `qwen2.5:0.5b` a veces clasifica incorrectamente.
- **Sin autenticación en la API**: cualquier cliente puede llamar a los endpoints.
- **Sin HTTPS**: el tráfico viaja en texto plano. En producción se usaría TLS.
- **Una sola instancia de la API**: no hay balanceador de carga ni escalado horizontal.
- **Base de datos sin réplica**: si PostgreSQL falla, no hay failover automático.
