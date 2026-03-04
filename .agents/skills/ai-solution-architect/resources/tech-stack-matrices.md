# Matrices de Selección de Tech Stack

## 1. MODELOS LLM — Comparativa Detallada (2025)

### Modelos Comerciales (API)

| Modelo | Context Window | Fortaleza | Debilidad | Costo (input/output por 1M tokens) | Mejor para |
|--------|---------------|-----------|-----------|--------------------------------------|-----------|
| Claude Opus 4 | 200K | Razonamiento complejo, instrucciones largas | Más lento, más caro | ~$15/$75 | Análisis profundo, tareas complejas |
| Claude Sonnet 4 | 200K | Balance calidad/velocidad/costo | No el mejor en razonamiento puro | ~$3/$15 | Producción general, RAG |
| Claude Haiku 4 | 200K | Ultra-rápido, ultra-barato | Menos capaz en tareas complejas | ~$0.25/$1.25 | Clasificación, routing, alto volumen |
| GPT-4o | 128K | Multi-modal fuerte, ecosistema amplio | Costo variable, rate limits | ~$2.50/$10 | Multi-modal, ecosistema OpenAI |
| GPT-4o-mini | 128K | Muy barato, rápido | Menos capaz | ~$0.15/$0.60 | Alto volumen, tareas simples |
| Gemini 2.5 Pro | 1M+ | Context window enorme | Consistency variable | ~$1.25/$5 | Documentos muy largos |
| Gemini 2.5 Flash | 1M+ | Rápido, context grande | Menos preciso | ~$0.075/$0.30 | Procesamiento masivo |

### Modelos Open Source (self-hosted)

| Modelo | Parámetros | VRAM mínima | Fortaleza | Mejor para |
|--------|-----------|-------------|-----------|-----------|
| Llama 3.3 70B | 70B | 40GB (4-bit) | General purpose, fuerte | Producción on-premise |
| Llama 3.2 8B | 8B | 6GB (4-bit) | Eficiente, rápido | Edge, dispositivos |
| Mistral Large 2 | 123B | 80GB (4-bit) | Multilingüe, código | Enterprise on-premise |
| Mixtral 8x7B | 46.7B (MoE) | 24GB (4-bit) | Balance rendimiento/costo | MoE, cost-effective |
| Qwen 2.5 72B | 72B | 40GB (4-bit) | Multilingüe (esp. asiático) | Mercados asiáticos |
| DeepSeek-V3 | 671B (MoE) | Cluster | Razonamiento, código | Research, tareas complejas |

### Decisión: API vs Self-hosted

| Factor | API (comercial) | Self-hosted (open source) |
|--------|----------------|--------------------------|
| Setup | Minutos | Días-semanas |
| Costo a bajo volumen | Más barato | Más caro (infra fija) |
| Costo a alto volumen | Más caro | Más barato |
| Privacidad | Datos salen a tercero | Datos en tu infra |
| Latencia | Variable (red) | Predecible (local) |
| Mantenimiento | Cero | Alto (updates, GPU, monitoring) |
| Calidad | Frontier models | Ligeramente inferior (pero mejorando) |

**Recomendación:** API para MVP y volumen bajo-medio. Self-hosted solo si
privacidad es requisito hard o volumen justifica inversión en infra.

---

## 2. VECTOR DATABASES — Comparativa Detallada

| Criterio | Pinecone | Qdrant | Weaviate | Chroma | pgvector | Milvus |
|----------|----------|--------|----------|--------|----------|--------|
| **Tipo** | Managed SaaS | OSS + Cloud | OSS + Cloud | OSS (lightweight) | PG extension | OSS + Cloud |
| **Setup** | 5 min | 15 min (Docker) | 15 min (Docker) | 5 min (pip) | Si ya tienes PG: 5 min | 30 min |
| **Escalabilidad** | Excelente | Muy buena | Buena | Limitada | Buena (con PG) | Excelente |
| **Búsqueda híbrida** | ✅ Sparse+Dense | ✅ BM25+Vector | ✅ Nativa | ❌ Solo vector | ❌ (necesita FTS separado) | ✅ |
| **Filtering** | Bueno | Excelente | Excelente | Básico | SQL nativo (excelente) | Bueno |
| **Multi-tenancy** | ✅ Namespaces | ✅ Collections | ✅ Nativo | ❌ | ✅ (schemas/rows) | ✅ |
| **Costo (1M vectors)** | ~$70/mes | Self-host: infra | Self-host: infra | Gratis (local) | Gratis (en PG existente) | Self-host: infra |
| **Costo (cloud)** | $70+/mes | Desde $25/mes | Desde $25/mes | N/A | N/A | Zilliz desde $65/mes |
| **Mejor para** | SaaS, escala, zero-ops | Producción, control, hybrid | Multi-modal, enterprise | Prototipo, dev local | Si ya usas PostgreSQL | Big data, enterprise |

### Cuándo elegir cada uno

```
¿Ya usas PostgreSQL en producción?
├─ SÍ y <5M vectores → pgvector (no agregues complejidad)
├─ SÍ pero >5M vectores → Qdrant o Pinecone (PG no escala bien para esto)
└─ NO →
    ¿Presupuesto para managed service?
    ├─ SÍ → Pinecone (zero-ops) o Qdrant Cloud
    └─ NO →
        ¿Prototipo o producción?
        ├─ Prototipo → Chroma (más simple)
        └─ Producción → Qdrant self-hosted (mejor balance)
```

---

## 3. FRAMEWORKS DE ORQUESTACIÓN

| Criterio | LangChain | LlamaIndex | Haystack | Semantic Kernel | Custom |
|----------|-----------|------------|----------|-----------------|--------|
| **Enfoque** | General-purpose chains | Data-centric RAG | Search/NLP pipelines | Microsoft ecosystem | Tu código |
| **RAG nativo** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐ (construyes tú) |
| **Agentes** | ⭐⭐⭐⭐ (LangGraph) | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐ (construyes tú) |
| **Abstracción** | Alta (muchas capas) | Media-alta | Media | Alta | Ninguna |
| **Flexibilidad** | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Curva aprendizaje** | Empinada | Moderada | Moderada | Moderada | Alta (pero predecible) |
| **Producción ready** | ⭐⭐⭐ (mejorado con LangGraph) | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Debugging** | Difícil (muchas abstracciones) | Medio | Bueno | Bueno | Excelente |
| **Lock-in risk** | Alto | Medio | Bajo | Alto (Microsoft) | Ninguno |

### Cuándo elegir cada uno

```
¿Qué necesitas?
├─ RAG es el caso principal
│   └─ LlamaIndex (mejor indexing y retrieval nativo)
├─ Agentes + herramientas + flujos complejos
│   └─ LangGraph (estado + grafos + persistencia)
├─ Ecosistema Microsoft (Azure, C#)
│   └─ Semantic Kernel
├─ NLP pipeline (no solo LLM)
│   └─ Haystack
└─ Control total, equipo experimentado
    └─ Custom (Python + httpx + instructor)
```

**Nota importante:** Muchos equipos senior prefieren implementación custom usando
las APIs directamente con `instructor` (structured outputs) y `httpx` (HTTP client).
Los frameworks agregan abstracción que puede estorbar en producción.

---

## 4. MODELOS DE EMBEDDING

| Modelo | Dimensiones | Multilingüe | Costo | Fortaleza |
|--------|-------------|-------------|-------|-----------|
| text-embedding-3-large (OpenAI) | 3072 (reducible) | ✅ | $0.13/1M tokens | Mejor calidad general |
| text-embedding-3-small (OpenAI) | 1536 (reducible) | ✅ | $0.02/1M tokens | Balance costo/calidad |
| Voyage-3-large | 1024 | ✅ | $0.18/1M tokens | Excelente para código y técnico |
| Voyage-3-lite | 512 | ✅ | $0.02/1M tokens | Ultra-barato, buena calidad |
| Cohere embed-v4 | 1024 | ✅ (100+ langs) | $0.10/1M tokens | Multimodal, compression |
| BGE-M3 (open source) | 1024 | ✅ (100+ langs) | Gratis (self-host) | Sin vendor lock, on-premise |
| E5-mistral-7b-instruct (OSS) | 4096 | ✅ | Gratis (self-host) | Alta calidad, instruction-following |
| nomic-embed-text (OSS) | 768 | ✅ | Gratis (self-host) | Ligero, Matryoshka dims |

### Decisión de embedding

```
¿Privacidad es requisito hard?
├─ SÍ → BGE-M3 o nomic-embed-text (self-hosted)
└─ NO →
    ¿Presupuesto?
    ├─ Muy limitado → text-embedding-3-small o Voyage-3-lite
    ├─ Moderado → text-embedding-3-large o Cohere embed-v4
    └─ Sin restricción → Voyage-3-large (código) o text-embedding-3-large (general)
```

---

## 5. HERRAMIENTAS DE PARSING/INGESTA

| Herramienta | Formatos | Fortaleza | Cuándo usar |
|------------|---------|-----------|-------------|
| Unstructured.io | PDF, DOCX, HTML, PPT, imágenes | General-purpose, muchos formatos | Default para la mayoría |
| Marker | PDF → Markdown | Mejor calidad para PDFs académicos | Papers, documentos complejos |
| Docling (IBM) | PDF, DOCX | Tablas y estructura | Documentos con tablas |
| LlamaParse (LlamaIndex) | PDF, DOCX | Integrado con LlamaIndex | Si ya usas LlamaIndex |
| PyPDF / pdfplumber | PDF | Ligero, rápido | PDFs simples, texto plano |
| Apache Tika | Todo formato | Enterprise, JVM | Si ya tienes infra Java |

---

## 6. OBSERVABILIDAD Y EVALUACIÓN

| Herramienta | Tipo | Costo | Fortaleza |
|------------|------|-------|-----------|
| LangSmith | Tracing + eval | Freemium ($39+/mes) | Nativo LangChain, buen UI |
| LangFuse | Tracing + eval | OSS + Cloud | Open source, flexible |
| Arize Phoenix | Tracing + eval | OSS | Excelente para RAG eval |
| Weights & Biases | Experiment tracking | Freemium | ML experiments, fine-tuning |
| RAGAS | Evaluation framework | OSS (gratis) | Standard para RAG evaluation |
| Braintrust | Eval + tracing | Freemium | Simple, buena UX |

### Stack de observabilidad recomendado

**MVP:** LangFuse (OSS) + RAGAS
**Producción:** LangSmith o Arize Phoenix + RAGAS + custom dashboards
**Enterprise:** Datadog/Grafana + LangSmith + custom eval pipeline
