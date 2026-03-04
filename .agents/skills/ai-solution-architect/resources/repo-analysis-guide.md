# Guía de Análisis de Repositorios — Referencia Detallada

## 1. QUÉ BUSCAR EN UN REPOSITORIO

### Estructura de archivos (pistas arquitectónicas)

| Patrón de directorios | Arquitectura probable |
|----------------------|----------------------|
| `src/`, `lib/`, `main.py` | Monolito simple |
| `services/`, `microservices/` | Microservicios |
| `api/`, `frontend/`, `backend/` | Full-stack separado |
| `functions/`, `handlers/` | Serverless / FaaS |
| `agents/`, `skills/`, `tools/` | Sistema de agentes IA |
| `pipeline/`, `dags/`, `flows/` | Data pipeline / ETL |
| `models/`, `training/`, `inference/` | ML/AI con ciclo de vida |
| `embeddings/`, `vectorstore/`, `retrieval/` | Sistema RAG |
| `docker-compose.yml` + múltiples servicios | Multi-container |
| `terraform/`, `cdk/`, `pulumi/` | Infrastructure as Code |

### Archivos clave que revelan arquitectura

| Archivo | Qué revela |
|---------|-----------|
| `README.md` | Propósito, setup, arquitectura de alto nivel |
| `docker-compose.yml` | Servicios, dependencias entre containers, puertos |
| `package.json` / `requirements.txt` / `Cargo.toml` | Dependencias, frameworks |
| `.env.example` | Servicios externos (APIs, DBs, secrets) |
| `Dockerfile` | Runtime, sistema operativo, build process |
| `openapi.yaml` / `schema.graphql` | Contratos de API |
| `Makefile` / `justfile` | Comandos de desarrollo y deployment |
| `CLAUDE.md` / `.cursorrules` / `AGENTS.md` | Contexto para agentes de IA |
| `alembic/` / `migrations/` | Base de datos relacional con migraciones |
| `.github/workflows/` | CI/CD pipeline |

### Dependencias que revelan patrones

| Dependencia | Indica |
|-----------|--------|
| `langchain`, `llamaindex` | Orquestación LLM / RAG |
| `openai`, `anthropic`, `google-genai` | Uso de LLM via API |
| `chromadb`, `qdrant-client`, `pinecone` | Vector database (RAG) |
| `transformers`, `torch`, `tensorflow` | ML/DL local (fine-tuning, inference) |
| `fastapi`, `flask`, `express` | API backend |
| `celery`, `rq`, `dramatiq` | Procesamiento async / cola de tareas |
| `redis` | Cache y/o message broker |
| `sqlalchemy`, `prisma`, `drizzle` | ORM / base de datos relacional |
| `boto3`, `google-cloud-*` | Cloud services (AWS/GCP) |
| `playwright`, `selenium` | Automatización web / scraping |
| `streamlit`, `gradio` | UI rápida para demos de ML |

---

## 2. CÓMO MAPEAR COMPONENTES

### Paso 1: Identificar las "cajas"
Cada servicio, módulo principal, o directorio de primer nivel es una "caja" potencial.

### Paso 2: Identificar las "flechas"
- Imports entre módulos → dependencia interna
- HTTP calls / gRPC → comunicación entre servicios
- Message queues → comunicación asíncrona
- DB connections → dependencia de datos
- API keys en `.env` → servicios externos

### Paso 3: Clasificar por capa

| Capa | Indicadores |
|------|-----------|
| Presentación | `frontend/`, `pages/`, `components/`, `templates/`, `static/` |
| API/Gateway | `api/`, `routes/`, `endpoints/`, `controllers/` |
| Orquestación | `chains/`, `workflows/`, `orchestrator/`, `agents/` |
| Inteligencia | `llm/`, `models/`, `inference/`, `retrieval/`, `embeddings/` |
| Datos | `db/`, `migrations/`, `schemas/`, `vectorstore/`, `index/` |
| Infraestructura | `docker/`, `terraform/`, `k8s/`, `.github/workflows/` |

---

## 3. PATRONES COMUNES EN REPOS DE IA

### Patrón A: RAG Básico (Chatbot sobre documentos)

```
Estructura típica:
├── app.py (o main.py)         ← Entry point, API
├── ingest.py                   ← Pipeline de ingesta
├── retriever.py                ← Lógica de búsqueda
├── chain.py (o pipeline.py)    ← Cadena LLM
├── config.py                   ← Settings, API keys
├── data/                       ← Documentos fuente
└── vectorstore/                ← Índice persistido
```

Explicación no técnica: "Es como un asistente que primero busca en tu archivo
de documentos la información relevante, y luego redacta una respuesta basándose
en lo que encontró."

### Patrón B: Agente con herramientas

```
Estructura típica:
├── agent.py                    ← Agente principal
├── tools/                      ← Herramientas disponibles
│   ├── search.py
│   ├── calculator.py
│   └── database.py
├── prompts/                    ← Templates de instrucciones
└── memory/                     ← Historial de conversación
```

Explicación no técnica: "Es como un asistente que no solo responde preguntas,
sino que puede hacer cosas: buscar en internet, consultar bases de datos,
o ejecutar cálculos. Decide qué herramienta usar según lo que le pidas."

### Patrón C: Pipeline ML (entrenamiento + servicio)

```
Estructura típica:
├── data/                       ← Datos de entrenamiento
├── notebooks/                  ← Experimentación
├── training/                   ← Scripts de entrenamiento
│   ├── train.py
│   ├── evaluate.py
│   └── config.yaml
├── models/                     ← Modelos guardados
├── serving/                    ← API de inferencia
│   ├── app.py
│   └── predict.py
└── monitoring/                 ← Métricas en producción
```

Explicación no técnica: "Es como una escuela donde primero entrenas al asistente
con ejemplos (training), luego lo evalúas con un examen (evaluate), y finalmente
lo pones a trabajar atendiendo solicitudes (serving), mientras supervisas que siga
haciendo bien su trabajo (monitoring)."

### Patrón D: Multi-agente

```
Estructura típica:
├── orchestrator.py             ← Coordinador
├── agents/                     ← Agentes especializados
│   ├── researcher.py
│   ├── writer.py
│   └── reviewer.py
├── shared/                     ← Memoria compartida
│   ├── state.py
│   └── tools.py
└── config/                     ← Configuración de flujos
```

Explicación no técnica: "Es como una oficina con empleados especializados:
un investigador busca información, un redactor escribe, y un revisor verifica.
Un gerente (orquestador) les asigna tareas y coordina el resultado final."

---

## 4. NIVELES DE EXPLICACIÓN

### Nivel 1: Ejecutivo (1 párrafo)
"Este sistema [hace qué] para [quién]. Funciona [analogía simple].
El beneficio principal es [valor de negocio]."

### Nivel 2: Gerencial (5-10 líneas + diagrama simplificado)
Agrega: componentes principales como cajas con flechas,
flujo de usuario paso a paso, costos/riesgos principales.

### Nivel 3: Técnico-gerencial (1-2 páginas + diagramas)
Agrega: stack tecnológico, decisiones arquitectónicas,
trade-offs, oportunidades de mejora.

### Nivel 4: Técnico completo (documento de arquitectura)
Todo lo anterior + ADRs, diagramas de secuencia, estimaciones de costo,
configuraciones, deployment patterns.

---

## 5. ERRORES COMUNES AL EXPLICAR ARQUITECTURA

| Error | Problema | Solución |
|-------|----------|----------|
| Dump de tecnologías | "Usa FastAPI con Qdrant y LangChain" no dice nada a no-técnicos | Explica el PARA QUÉ antes del CON QUÉ |
| Asumir conocimiento | "Implementa RAG con reranking" | Explica cada concepto con analogía |
| Diagrama sobrecargado | 20 cajas con flechas en todas direcciones | Máximo 5-7 cajas para no-técnicos |
| Foco en cómo, no en qué | Explicar la implementación sin el problema | Empieza SIEMPRE por el problema de negocio |
| Sin escala humana | "Procesa 10M tokens" no significa nada | "Puede leer el equivalente a 50 libros en segundos" |
