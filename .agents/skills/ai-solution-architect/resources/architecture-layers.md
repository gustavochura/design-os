# Capas Arquitectónicas — Referencia Detallada

## CAPA 1: PRESENTACIÓN

### Responsabilidad
Interfaz entre el usuario y el sistema de IA. Gestiona la experiencia de usuario,
autenticación, y presentación de resultados.

### Componentes típicos

| Componente | Función | Opciones |
|-----------|---------|----------|
| Web App | UI principal | Next.js, React, Vue, Streamlit (prototipo) |
| Mobile App | UI móvil | React Native, Flutter, Swift/Kotlin nativo |
| Chat Widget | Interfaz conversacional | Custom, Intercom, widget embebido |
| API REST/GraphQL | Para integraciones B2B | FastAPI, Express, Go |
| Slack/Teams Bot | Canal empresarial | Bolt (Slack), Bot Framework (Teams) |

### Decisiones clave
- ¿Interfaz conversacional o formulario estructurado?
- ¿Streaming de respuestas (SSE/WebSocket) o request-response?
- ¿Autenticación requerida? ¿SSO?
- ¿Multi-tenant o single-tenant?

### Patrón de streaming (recomendado para LLMs)
```
User → Frontend → API (SSE stream) → Orchestrator → LLM (streaming)
         ↑                                                    │
         └────────────── tokens incrementales ←────────────────┘
```

---

## CAPA 2: ORQUESTACIÓN

### Responsabilidad
Coordina el flujo de trabajo entre componentes. Es el "cerebro" del sistema
que decide qué hacer con cada request.

### Componentes típicos

| Componente | Función | Opciones |
|-----------|---------|----------|
| API Gateway | Entrada, rate limiting, auth | Kong, AWS API Gateway, nginx |
| Orchestrator | Lógica de flujo | LangChain, LlamaIndex, LangGraph, custom |
| Session Manager | Estado de conversación | Redis, DynamoDB, PostgreSQL |
| Memory Store | Historial contextual | Redis (buffer), Vector DB (long-term) |
| Queue / Event Bus | Procesamiento async | RabbitMQ, SQS, Kafka (alto volumen) |
| Cache | Respuestas frecuentes | Redis, Memcached |

### Patrones de orquestación

**Simple Chain (90% de los casos):**
```
Query → Retrieve → Augment Prompt → Generate → Post-process → Response
```

**Router Pattern (múltiples pipelines):**
```
Query → Router (clasifica intención)
          ├─ FAQ → Cache lookup → Response
          ├─ Documento → RAG pipeline → Response
          ├─ Acción → Agent pipeline → Response
          └─ Chitchat → LLM directo → Response
```

**Agent Loop (cuando se necesitan acciones):**
```
Query → Plan → Execute Tool → Observe Result → Reason
          ↑                                       │
          └──── ¿Tarea completa? NO ──────────────┘
                                 SÍ → Response
```

### Decisiones clave
- ¿Sync o async? (sync para <5s, async + webhook para tareas largas)
- ¿Framework o custom? (framework para MVP, custom para control total)
- ¿Retry policy? (exponential backoff para APIs de LLM)
- ¿Circuit breaker? (fallback si LLM provider falla)

---

## CAPA 3: INTELIGENCIA

### Responsabilidad
Contiene la lógica de IA: retrieval, generación, y post-procesamiento.
Es donde vive el "cerebro" inteligente del sistema.

### Sub-capa 3A: Retrieval Pipeline (si RAG)

| Paso | Función | Opciones |
|------|---------|----------|
| Query Processing | Limpiar, expandir query | Query rewriting, HyDE, multi-query |
| Retrieval | Buscar chunks relevantes | Vector search, BM25, híbrido |
| Reranking | Re-ordenar por relevancia | Cohere Rerank, Cross-encoder, ColBERT |
| Context Assembly | Construir prompt con contexto | Stuffing, map-reduce, refine |

**Pipeline de retrieval progresivo:**
```
Nivel 1 (Básico):     Query → Vector Search → Top-K → LLM
Nivel 2 (Intermedio):  Query → Hybrid Search → Rerank → Top-K → LLM
Nivel 3 (Avanzado):    Query Rewrite → Multi-source Search → Rerank →
                       Contextual Compression → Self-check → LLM
```

### Sub-capa 3B: LLM Gateway

| Componente | Función |
|-----------|---------|
| Model Router | Selecciona modelo según tarea (Haiku para simple, Opus para complejo) |
| Prompt Builder | Construye prompt con template + contexto + historial |
| Token Counter | Gestiona context window (trunca si necesario) |
| Fallback | Cambia provider si el primario falla |
| Cost Tracker | Registra costo por request |

**Patrón multi-modelo (optimización de costo):**
```
Request → Complexity Classifier
            ├─ Simple → Claude Haiku ($0.25/M input)
            ├─ Medium → Claude Sonnet ($3/M input)
            └─ Complex → Claude Opus ($15/M input)
```

### Sub-capa 3C: Post-processing y Guardrails

| Guardrail | Función | Implementación |
|-----------|---------|----------------|
| Input filter | Bloquea prompt injection, contenido dañino | Regex + clasificador |
| Grounding check | Verifica que respuesta se basa en contexto | NLI model o LLM-as-judge |
| Output filter | Bloquea PII, contenido inapropiado | Regex + clasificador |
| Citation formatter | Agrega referencias a las fuentes | Mapeo chunk → source |
| Format enforcer | Asegura formato de salida (JSON, markdown) | Structured output o parser |

**Multi-layer guardrails (obligatorio para producción):**
```
Input → [Filter 1: regex] → [Filter 2: classifier] → LLM →
     → [Check 1: grounding] → [Check 2: output filter] → Response
```

---

## CAPA 4: DATOS E INDEXACIÓN

### Responsabilidad
Gestiona el ciclo de vida de los datos: ingesta, procesamiento, embedding,
almacenamiento y actualización.

### Pipeline de Ingesta

```
Fuentes              Extracción         Procesamiento       Indexación
┌──────────┐         ┌──────────┐       ┌──────────┐       ┌──────────┐
│ PDFs     │────────▶│ Parser   │──────▶│ Chunking │──────▶│ Embed    │
│ Docs     │         │ (Unstruc-│       │ + Clean  │       │ + Store  │
│ Web      │         │  tured,  │       │          │       │ in Vector│
│ DBs      │         │  Marker) │       │          │       │ DB       │
│ APIs     │         └──────────┘       └──────────┘       └──────────┘
└──────────┘
```

### Estrategias de Chunking (detalle)

**Fixed-size overlapping:**
- Chunks de N tokens con overlap de M tokens
- Simple, predecible, funciona bien para documentos homogéneos
- Chunk: 512 tokens, Overlap: 50 tokens (típico)

**Semantic chunking:**
- Divide por cambios semánticos (embeddings de oraciones consecutivas)
- Mejor para documentos con múltiples temas
- Más costoso computacionalmente

**Hierarchical / Parent-Child:**
- Padre: sección completa (2048 tokens)
- Hijo: párrafos individuales (256 tokens)
- Busca por hijo, retorna padre como contexto
- Mejor balance precisión/contexto

**Document-structured:**
- Respeta estructura del documento (headers, secciones)
- Ideal para documentos con estructura clara (papers, manuales)
- Requiere parsing inteligente

### Decisiones clave
- ¿Frecuencia de actualización? (real-time, batch diario, manual)
- ¿Formatos de fuente? (determina parsers necesarios)
- ¿Metadata a preservar? (autor, fecha, sección, página)
- ¿Estrategia de deduplicación?
- ¿Versionamiento de índices? (para rollback)

---

## CAPA 5: INFRAESTRUCTURA

### Responsabilidad
Compute, storage, networking, monitoring, CI/CD, seguridad.

### Componentes por entorno

| Componente | Desarrollo | Staging | Producción |
|-----------|-----------|---------|------------|
| Compute | Local / Docker | Cloud (small) | Cloud (auto-scale) |
| Vector DB | Chroma (local) | Qdrant/Pinecone (managed) | Managed + replica |
| LLM | API directa | API con cache | API con fallback + cache |
| Monitoring | Console logs | Grafana básico | Full stack (Datadog/Grafana) |
| CI/CD | Manual | GitHub Actions | Full pipeline con eval gates |

### Métricas clave a monitorear

| Categoría | Métrica | Target |
|-----------|---------|--------|
| Latencia | P50, P95, P99 response time | <2s P50, <5s P95 |
| Calidad | Faithfulness, relevancy score | >0.85 faith, >0.80 relevancy |
| Costo | Costo por query, por usuario, por día | Dentro de presupuesto |
| Uso | Queries/día, usuarios activos | Crecimiento esperado |
| Errores | Tasa de error, timeouts, fallbacks | <1% error rate |
| Retrieval | Context precision, recall | >0.75 precision |

### Patrones de deployment

**Blue-Green (recomendado para índices):**
- Blue: índice actual en producción
- Green: nuevo índice con datos actualizados
- Switch tráfico cuando Green pasa evaluación

**Canary (recomendado para cambios de modelo/prompt):**
- 5% → 25% → 50% → 100% de tráfico
- Rollback automático si métricas degradan

---

## VARIANTES ARQUITECTÓNICAS

### Variante A: RAG Simple (MVP)
```
User → API → LangChain/LlamaIndex → Vector DB → LLM API → Response
```
Componentes: 4-5 | Timeline: 2-3 semanas | Equipo: 1-2 devs

### Variante B: RAG Producción
```
User → CDN → API Gateway → Orchestrator → Cache
                                ├─ Retrieval (hybrid + rerank)
                                ├─ LLM Gateway (multi-model)
                                └─ Guardrails (multi-layer)
       Monitoring ← Logging ← Eval Pipeline
```
Componentes: 10-15 | Timeline: 2-3 meses | Equipo: 3-5 devs

### Variante C: Plataforma AI (Enterprise)
```
User → Load Balancer → API Gateway → Router
                                       ├─ RAG Pipeline
                                       ├─ Agent Pipeline
                                       ├─ Fine-tuned Model
                                       └─ Hybrid Pipeline
       Admin Dashboard → Eval System → A/B Testing
       Data Pipeline → ETL → Multi-source Indexing
```
Componentes: 20+ | Timeline: 4-8 meses | Equipo: 5-10+
