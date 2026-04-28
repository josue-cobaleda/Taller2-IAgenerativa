# Fase 1 — Selección y justificación de componentes del sistema RAG

## Contexto y restricciones de diseño

Este taller extiende el prototipo desarrollado en el **Taller Práctico #1**, donde se implementó un retrieval rudimentario con `NumPy` y similitud coseno sobre embeddings generados con Sentence Transformers, usando **LLaMA 3 vía Ollama** como LLM de generación.

Para el Taller 2 se formaliza esa arquitectura con componentes estándar de la industria, manteniendo dos decisiones heredadas del Taller 1 que condicionan las elecciones de esta fase:

1. **LLM local (Ollama + LLaMA 3)** → elegido por privacidad, costo cero en inferencia y control total sobre el stack. Esto favorece componentes open-source y autocontenidos.
2. **Caso de uso en español** → EcoMarket opera en mercado hispanohablante, por lo que la capacidad multilingüe del modelo de embeddings es un criterio de primer nivel.

Los criterios de evaluación para cada componente son, en orden:

| Criterio | Peso | Razón |
|---|---|---|
| Rendimiento en español | Alto | Idioma principal del dataset y los usuarios |
| Costo | Alto | EcoMarket es una empresa en crecimiento, no una BigTech |
| Privacidad | Alto | Los documentos contienen datos internos y de clientes |
| Escalabilidad | Medio | Crítico a futuro, no bloqueante para el prototipo |
| Facilidad de integración | Medio | Acelera el tiempo a producción |
| Madurez del ecosistema | Medio | Integración con LangChain, comunidad activa |

---

## Decisión 1 — Modelo de embeddings

### Opciones consideradas

| Modelo | Tipo | Idiomas | Dims | Costo | Observación |
|---|---|---|---|---|---|
| `intfloat/multilingual-e5-large` | Open-source (HF) | 100+ (fuerte en español) | 1024 | $0 | Estado del arte multilingüe open-source, benchmark MTEB sólido |
| `BAAI/bge-m3` | Open-source (HF) | 100+ | 1024 | $0 | Muy competitivo, soporta long context (8K) y dense+sparse |
| `sentence-transformers/paraphrase-multilingual-mpnet-base-v2` | Open-source (HF) | 50+ | 768 | $0 | Más ligero pero calidad inferior en español técnico |
| `OpenAI text-embedding-3-small` | Propietario (API) | Multilingüe | 1536 | ~$0.02 / 1M tokens | Excelente calidad, pero dependencia cloud y envío de datos a terceros |
| `OpenAI text-embedding-3-large` | Propietario (API) | Multilingüe | 3072 | ~$0.13 / 1M tokens | Máxima calidad, costo alto para indexación masiva |
| `Cohere embed-multilingual-v3.0` | Propietario (API) | 100+ | 1024 | Pago por uso | Competitivo en español, misma desventaja de dependencia cloud |

### Decisión: `intfloat/multilingual-e5-large`

### Justificación

**Rendimiento en español.** El modelo E5 multilingüe fue entrenado específicamente para recuperación semántica en múltiples idiomas y figura entre los primeros puestos de MTEB (Massive Text Embedding Benchmark) para tareas de retrieval en español. Supera claramente al `MiniLM` usado en el Taller 1 cuando los chunks superan las 100–150 palabras, que es el caso de las políticas de EcoMarket.

**Costo.** Al ser open-source y ejecutarse localmente, el costo marginal por documento indexado o por query es **cero**. Esto es crítico porque una empresa con miles de consultas diarias y un catálogo que crece, usando un modelo propietario, acumularía costos de embeddings tanto en la fase de ingesta (reindexar cuando cambian políticas o llegan productos nuevos) como en cada consulta del usuario (embedding de la pregunta).

**Privacidad.** Todas las políticas internas, el catálogo y las FAQ de EcoMarket se mantienen dentro de la infraestructura propia. No se envía contenido a servidores externos, lo cual es consistente con el análisis ético del Taller 1 y reduce el riesgo de fuga de información sensible de clientes.

**Integración con el stack existente.** Se carga trivialmente desde `langchain-huggingface` con `HuggingFaceEmbeddings`, sin cambios en el modelo de LLM ya probado en el Taller 1.

**Trade-off aceptado.** El modelo pesa ~2.2 GB y requiere ~4 GB de VRAM o RAM para inferencia eficiente. Esto es asumible en la infraestructura local (hardware disponible: GeForce RTX 4050). Si este requisito fuera bloqueante, la alternativa inmediata sería `multilingual-e5-base` (más liviano, ligeramente inferior en calidad).

### Camino de evolución

Para producción con volúmenes altos, se evaluaría migrar a `BAAI/bge-m3` (mejor soporte de contexto largo) o a un modelo propietario con contrato de privacidad adecuado (ej. OpenAI con Zero Data Retention) si el incremento en calidad justifica el costo recurrente.

---

## Decisión 2 — Base de datos vectorial

### Opciones consideradas

| Base vectorial | Modelo | Costo | Escalabilidad | Facilidad | Privacidad | Ecosistema LangChain |
|---|---|---|---|---|---|---|
| **ChromaDB** | Open-source, embedded | Gratis | Media (hasta ~1M vectores cómodos) | Muy alta (`pip install`) | Total (local) | Integración nativa |
| **Pinecone** | SaaS cloud | Free tier → pago por uso | Muy alta (miles de millones) | Alta (API) | Depende del proveedor | Integración nativa |
| **Weaviate** | Open-source + cloud | Gratis self-hosted, pago cloud | Alta | Media (Docker, schema) | Configurable | Integración nativa |
| **Qdrant** | Open-source + cloud | Gratis self-hosted, pago cloud | Alta | Media (Docker) | Configurable | Integración nativa |
| **FAISS** | Librería in-memory | Gratis | Alta en memoria | Alta | Total (local) | Integración nativa pero sin persistencia automática |

### Decisión

- **Prototipo (este taller):** **ChromaDB** con persistencia local.
- **Producción (recomendación para EcoMarket a futuro):** **Pinecone** o **Weaviate cloud**.

### Justificación

**Por qué ChromaDB para el prototipo:**

- **Facilidad extrema.** Se instala con `pip install chromadb` y no requiere servidor externo, Docker ni configuración de cluster. Para un taller académico y un primer MVP, esto reduce la fricción de integración a casi cero y permite enfocar el esfuerzo en la calidad del retrieval y del prompt, no en DevOps.
- **Persistencia local.** Los vectores se guardan en disco (`./chroma_db/`), lo que significa que la indexación se hace una sola vez y las consultas posteriores no tienen que regenerar embeddings. Esto replica el comportamiento de una base vectorial de producción sin la complejidad operativa.
- **Consistencia con el stack.** El Taller 1 usa Ollama local y Sentence Transformers local. Añadir una base vectorial cloud rompería esa coherencia arquitectónica y añadiría dependencias de red innecesarias para la escala del caso.
- **Costo $0.** Coherente con el criterio de costo que domina las decisiones de esta fase.
- **Escala suficiente para EcoMarket prototipo.** Con políticas, catálogo y FAQ el volumen se mide en cientos de chunks, no en millones. ChromaDB opera sin problema en ese rango.

**Por qué Pinecone (o Weaviate cloud) para producción:**

Cuando EcoMarket escale a catálogos de decenas de miles de productos, historiales de interacciones y documentación multi-país, ChromaDB embedded deja de ser óptimo. En ese momento se necesita:

- **Separación compute/storage**, para que múltiples servicios (chat web, app móvil, backoffice) compartan el mismo índice.
- **Alta disponibilidad y réplicas**, para no perder el servicio si el proceso de la aplicación cae.
- **Búsqueda híbrida nativa** (vectorial + filtros por metadata), para poder restringir consultas por categoría, región o stock.

**Pinecone** destaca por simplicidad operativa (SaaS totalmente gestionado, se usa como una API) y escalabilidad probada. **Weaviate** es la alternativa si EcoMarket prefiere no depender de un vendor y puede asumir el self-hosting.

### Qué se descarta y por qué

- **FAISS puro** → no tiene persistencia gestionada ni metadata filtering nativo; exige más código para funcionar como base de conocimiento consultable por múltiples documentos con atributos.
- **Pinecone desde el prototipo** → agrega dependencia cloud, latencia de red y la necesidad de gestionar API keys para un caso donde el volumen aún no lo justifica. Overkill para el MVP.

---

## Arquitectura resultante

```
┌─────────────────┐       ┌──────────────────┐       ┌──────────────┐
│   Documentos    │──────▶│  Text Splitter   │──────▶│  Embeddings  │
│ (md, json, csv) │       │  (chunking)      │       │  e5-large    │
└─────────────────┘       └──────────────────┘       └──────┬───────┘
                                                            │
                                                            ▼
                                                   ┌────────────────┐
                                                   │   ChromaDB     │
                                                   │  (persistente) │
                                                   └────────┬───────┘
                                                            │
 Consulta usuario ──▶ Embedder ──▶ Retriever (top-k) ──────┘
                                         │
                                         ▼
                          Prompt con contexto recuperado
                                         │
                                         ▼
                           LLaMA 3 vía Ollama ──▶ Respuesta final
```

---

## Resumen de decisiones

| Componente | Elección | Racional principal |
|---|---|---|
| Modelo de embeddings | `intfloat/multilingual-e5-large` | Mejor calidad multilingüe open-source, costo cero, privacidad total |
| Base vectorial (prototipo) | ChromaDB local persistente | Cero fricción, consistente con stack local del Taller 1 |
| Base vectorial (producción) | Pinecone o Weaviate cloud | Escalabilidad, HA y búsqueda híbrida cuando el volumen lo exija |
| LLM | LLaMA 3 vía Ollama (heredado) | Privacidad, costo cero, control del stack |

Estas decisiones maximizan **privacidad** y **costo marginal cero** en el prototipo, a cambio de un techo de escala más bajo que se resolverá migrando la base vectorial (y eventualmente el modelo de embeddings) cuando EcoMarket lo requiera — sin tener que rediseñar el resto del sistema, gracias a la abstracción que provee LangChain sobre ambos componentes.