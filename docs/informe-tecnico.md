# Caracterización del modelo local

## Ficha del modelo

| Dato | Valor |
| :--- | :--- |
| Perfil de hardware | B |
| RAM total del equipo | 8 GB |
| Modelo y etiqueta | qwen2.5:0.5b |
| Tamaño en disco | 397 MB |
| Latencia 1 (ms) | 3531 |
| Latencia 2 (ms) | 2096 |
| Latencia 3 (ms) | 2377 |
| Latencia 4 (ms) | 2277 |
| Latencia 5 (ms) | 3020 |
| Latencia promedio (ms) | 2660 |
| RAM usada durante inferencia | 1.8 Gi (estimado) |
| Calidad percibida (1 a 5) | 4 - Responde bien y da la respuesta esperada|


# Plan de pruebas del despliegue

## Resultados

| ID | Tipo | Qué se verifica | Resultado esperado | Obtenido | Estado |
| :--- | :--- | :--- | :--- | :--- | :--- |
| P-01 | Funcional | GET /health responde | Código 200 y estado ok | 200 - ok | OK |
| P-02 | Funcional | POST /clasificar con motor eco | Código 200 y tipo correcto | 200 - fix | OK |
| P-03 | Funcional | Motor inválido | Código 400 | 400 | OK |
| P-04 | Acceso | Rol app_ia intenta DROP TABLE | Error de permisos | permission denied | OK |
| P-05 | Conectividad | La API resuelve el host db | Devuelve una IP interna | 172.18.0.2 | OK |
| P-06 | Disponibilidad | Reinicio del contenedor de BD | La API se recupera sola | ok tras 15s | OK |
| P-07 | Persistencia | down y up conservan los datos | Los registros siguen existiendo | sí | OK |
| P-08 | Carga | 10 usuarios sobre el motor eco | p95 < 800 ms y errores < 5% | p95=47ms, errores=0% | OK |
| P-09 | Caracterización | 10 inferencias con modelo | Promedio, mediana y p95 | promedio=2337ms, mediana=2210ms, p95=2686ms | OK |


## Análisis del cuello de botella

Comparando P-08 (motor eco bajo carga) con P-09 (motor ollama secuencial):

- El motor **eco** responde en **~31 ms** de promedio y soporta 10 usuarios concurrentes sin errores.
- El motor **ollama** responde en **~2337 ms** de promedio (75 veces más lento que el eco) y se ejecuta secuencialmente, sin concurrencia.

**El cuello de botella está en la inferencia del modelo**, no en la API ni en la base de datos. El tiempo se consume en:
1. La carga del modelo en memoria (por `OLLAMA_KEEP_ALIVE=0`, el modelo se descarga después de cada petición).
2. La generación de tokens, incluso cuando solo se pide una palabra.

El motor eco demuestra que la infraestructura (API + PostgreSQL + contenedores) es muy rápida: menos de 50 ms en el p95. Por lo tanto, el rendimiento del sistema está limitado por el modelo, no por la arquitectura.


## Propuestas de mejora

1. **Aumentar `OLLAMA_KEEP_ALIVE` a 5m** si el equipo tiene al menos 8 GB libres, para mantener el modelo en memoria entre peticiones y reducir la latencia a la mitad.
2. **Usar un modelo más grande** (`qwen2.5-coder:1.5b`) en equipos con 16 GB, mejorando la precisión sin sacrificar demasiado el rendimiento.
3. **Cachear clasificaciones frecuentes** para no invocar al modelo con textos repetidos: si el mismo mensaje llega dos veces, responder desde una caché.
4. **Escalar horizontalmente** la API con varias réplicas cuando aumente la demanda, y usar un balanceador de carga.
5. **Ejecutar el modelo en GPU** si el equipo lo permite, reduciendo la latencia a valores por debajo de 100 ms.
