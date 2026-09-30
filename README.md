# Clasificador de mensajes de commit

Servicio que clasifica mensajes de commit (feat, fix, docs, test, chore, refactor) usando un modelo de lenguaje local ejecutado con Ollama, expuesto por una API REST con FastAPI, con persistencia en PostgreSQL. Todo contenerizado con Docker y validado por un pipeline de CI en GitHub Actions.

## Integrantes y perfil de hardware

- **Katherine Sánchez** — Perfil B (8 GB RAM, SSD)
- **Mariana R.** — Perfil B (8 GB RAM, SSD)

## Requisitos mínimos

- Linux (nativo o WSL2 en Windows 10 2004+).
- Docker Engine y Docker Compose.
- Python 3.12.
- 4 GB de RAM mínimo, 15 GB de disco libre, virtualización habilitada.
- Ollama instalado en el host.

## Instalación paso a paso

### 1. Clonar el repositorio

```bash
git clone git@github.com:KathSRomero/clasificador-commits--MarianaR-y-KatherinS-.git
cd clasificador-commits--MarianaR-y-KatherinS-
