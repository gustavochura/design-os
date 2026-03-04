# Frameworks de Decisión Arquitectónica para Soluciones de IA

## 1. MATRIZ DE SELECCIÓN DE ENFOQUE

### Cuándo usar cada enfoque

**Prompt Engineering (LLM base sin customización)**
- ✅ El conocimiento general del LLM es suficiente
- ✅ Tareas genéricas: resumen, traducción, clasificación simple
- ✅ Prototipo rápido para validar concepto
- ✅ Presupuesto muy limitado
- ❌ Necesitas conocimiento específico del dominio
- ❌ Respuestas deben ser factuales sobre datos propios
- Tiempo a MVP: Días
- Costo: $ (solo API calls)

**RAG (Retrieval-Augmented Generation)**
- ✅ Datos propios que cambian frecuentemente
- ✅ Necesitas citabilidad (mostrar fuentes)
- ✅ Corpus de documentos grande (>100 docs)
- ✅ Precisión factual es crítica
- ✅ Equipo sin experiencia en ML puede implementar
- ❌ El problema requiere razonamiento complejo, no recuperación
- ❌ Datos muy estructurados (mejor SQL directo)
- Tiempo a MVP: 2-4 semanas
- Costo: $$ (API + vector DB + embedding)

**Fine-tuning**
- ✅ Necesitas tono/estilo/formato muy específico
- ✅ Tarea repetitiva con patrón claro (clasificación, extracción)
- ✅ Quieres reducir tokens de prompt (y costo)
- ✅ Latencia es crítica (modelo más pequeño fine-tuneado)
- ❌ No tienes datos de entrenamiento suficientes (<500 ejemplos)
- ❌ El conocimiento cambia frecuentemente
- ❌ No tienes infraestructura de ML
- Tiempo a MVP: 1-3 meses
- Costo: $$$ (compute de entrenamiento + hosting)

**Híbrido (RAG + Fine-tuning + Agentes)**
- ✅ Solución mission-critical de producción
- ✅ Múltiples tipos de tareas en un sistema
- ✅ Necesitas máxima calidad y control
- ✅ Presupuesto y equipo lo permiten
- ❌ MVP rápido (overengineering)
- ❌ Equipo sin experiencia en ML
- Tiempo a MVP: 2-6 meses
- Costo: $$$$ (todo lo anterior combinado)

---

## 2. ÁRBOL DE DECISIÓN RAG: TIPO DE RAG

```
¿Qué tipo de RAG necesitas?
│
├─ Consultas simples sobre documentos
│   └─ Naive RAG
│       Chunk → Embed → Retrieve top-k → Generate
│       Accuracy: ~0.55-0.65
│
├─ Consultas que requieren contexto de múltiples secciones
│   └─ Advanced RAG
│       + Query rewriting
│       + Hybrid search (BM25 + vector)
│       + Reranking (Cohere, Cross-encoder)
│       + Contextual embeddings
│       Accuracy: ~0.70-0.80
│
├─ Consultas complejas que requieren razonamiento
│   └─ Modular RAG
│       + Router (decide qué pipeline usar)
│       + Multi-step retrieval
│       + Self-reflection (verifica respuesta)
│       Accuracy: ~0.80-0.90
│
└─ Necesitas relaciones entre entidades
    └─ GraphRAG
        + Knowledge graph extraction
        + Graph-based retrieval
        + Entity resolution
        Accuracy: variable, mejor para relaciones
```

### Métricas de evaluación RAG

| Métrica | Qué mide | Target mínimo |
|---------|----------|---------------|
| Faithfulness | ¿La respuesta es fiel al contexto recuperado? | >0.85 |
| Answer Relevancy | ¿La respuesta es relevante a la pregunta? | >0.80 |
| Context Precision | ¿Los chunks recuperados son relevantes? | >0.75 |
| Context Recall | ¿Se recuperó toda la info necesaria? | >0.70 |
| Hallucination Rate | % de info fabricada | <5% |

Frameworks de evaluación: RAGAS, LangSmith, Phoenix (Arize)

---

## 3. DECISIÓN: ¿NECESITAS AGENTES?

```
¿Tu solución necesita agentes?
│
├─ ¿Necesita ejecutar acciones en sistemas externos?
│   ├─ NO → No necesitas agentes. RAG o prompt es suficiente.
│   └─ SÍ → ¿Cuántas herramientas?
│       ├─ 1-3 herramientas, flujo predecible
│       │   └─ Tool-calling simple (function calling del LLM)
│       ├─ 3-10 herramientas, flujo con decisiones
│       │   └─ Agente ReAct (Reason + Act loop)
│       └─ >10 herramientas o flujos paralelos
│           └─ Multi-agente (Manager + Workers)
│
├─ ¿Necesita planificación antes de ejecutar?
│   ├─ NO → Tool-calling directo
│   └─ SÍ → Plan-Execute pattern
│       1. LLM genera plan de pasos
│       2. Executor ejecuta cada paso
│       3. Reflector evalúa resultado
│       4. Re-planifica si necesario
│
└─ ¿Necesita memoria entre sesiones?
    ├─ NO → Stateless (más simple)
    └─ SÍ → Implementar memory layer
        ├─ Buffer memory (últimos N mensajes)
        ├─ Summary memory (resumen de conversación)
        ├─ Entity memory (entidades mencionadas)
        └─ Vector memory (búsqueda semántica sobre historial)
```

---

## 4. MATRIZ DE MADUREZ PROGRESIVA

| Etapa | Enfoque | Complejidad | Cuándo avanzar |
|-------|---------|-------------|----------------|
| 1. Prototipo | Prompt engineering | Baja | Cuando necesites datos propios |
| 2. MVP | Naive RAG | Media-baja | Cuando accuracy no sea suficiente |
| 3. Producto | Advanced RAG | Media | Cuando necesites acciones o personalización |
| 4. Plataforma | Hybrid (RAG + agents + fine-tune) | Alta | Solo si métricas lo justifican |

**Regla de oro:** Empieza en la etapa más baja que resuelva tu problema.
No saltes etapas. Cada etapa valida supuestos de la siguiente.

---

## 5. ANTI-PATRONES COMUNES

| Anti-patrón | Problema | Solución |
|-------------|----------|----------|
| RAG premature | Implementar RAG sin validar que prompt engineering no basta | Prueba primero con few-shot prompting |
| Context overload | Chunks demasiado grandes saturan el context window | Reduce chunk size, usa reranking para filtrar |
| Embedding mismatch | Modelo de embedding no alineado con dominio | Evalúa múltiples modelos con tus datos reales |
| Single-layer guardrails | Solo 1 capa de seguridad (90%+ bypass) | Multi-layer: input filter + output filter + grounding check |
| GPU por defecto | Asumir que necesitas GPU sin calcular | Calcula tokens/segundo necesarios. APIs pueden bastar. |
| Framework lock-in | Depender 100% de LangChain/LlamaIndex | Abstrae interfaces. El framework puede cambiar. |
| Ignoring evaluation | No medir calidad del RAG en producción | Implementa RAGAS o evaluación LLM-as-judge desde MVP |
| Over-chunking | Más chunks ≠ mejor retrieval | Optimiza chunk size con tus queries reales |
