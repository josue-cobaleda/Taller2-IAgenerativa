# Fase 3 — Integración y ejecución del código

## Objetivo

Llevar a la práctica las decisiones de las fases 1 y 2: ajustar el modelo del Taller Práctico #1 para que use el sistema RAG diseñado, integrando el LLM local (LLaMA 3.2 vía Ollama) con la base de conocimiento vectorial (ChromaDB persistente) y los documentos definidos (políticas, catálogo, FAQ). El sistema debe ser capaz de **responder a cualquier consulta del cliente o reconocer explícitamente cuándo no tiene información** para hacerlo, cumpliendo el enunciado del Taller 2.

La implementación completa vive en `Model-ollama-langchain.ipynb`. Este documento explica cómo el código del notebook materializa cada decisión textual de las fases anteriores.

---

## Mapa: del diseño al código

| Sección del notebook | Decisión que materializa | Documento de origen |
|---|---|---|
| **0. Instalación de dependencias** | Stack open-source local (LangChain + Ollama + HuggingFace + ChromaDB) | Fase 1 — criterios de costo y privacidad |
| **1. Carga de documentos** | Tres documentos indexados en RAG + pedidos como consulta directa por ID | Fase 2 — identificación de documentos |
| **2. Instalación de Ollama** | LLM local sin dependencias cloud | Fase 1 — heredado del Taller 1 |
| **3. LLM con LangChain + Ollama** | `llama3.2:3b` como generador | Fase 1 — privacidad y costo cero |
| **4. Vectorstore (ChromaDB)** | Embeddings `intfloat/multilingual-e5-large` + chunking diferenciado por tipo + persistencia local | Fase 1 (componentes) y Fase 2 (chunking) |
| **5. Carga de prompts** | Prompts en archivos planos para iterar sin tocar código | Heredado del Taller 1, ampliado con `prompt_general.txt` |
| **6. Herramienta directa de pedidos** | Pedidos NO indexados → consulta exacta por ID | Fase 2 — separación entre conocimiento estable y datos volátiles |
| **7. Pipeline RAG con router** | Router por regex (tracking → tool, todo lo demás → RAG) + fallback en el prompt | Decisión de implementación (Fase 3) |
| **8. Helper de impresión** | Trazabilidad de fuentes en cada respuesta | Mitigación del riesgo de alucinaciones (Taller 1, Fase 2) |
| **9. Pruebas** | Cuatro escenarios para validar el comportamiento bajo cada ruta | Rúbrica del Taller 2 |
| **10. Conclusiones** | Limitaciones y suposiciones del prototipo | Pedido explícito del enunciado |

---

## Flujo runtime

```
                   pregunta del cliente
                            │
                            ▼
              ┌─────────────────────────────┐
              │  router (regex 4 dígitos)   │
              └──────────────┬──────────────┘
                             │
            ┌────────────────┴───────────────┐
            │                                │
            ▼                                ▼
   ¿hay tracking number              cualquier otra
    válido?                          consulta
            │                                │
            ▼                                ▼
   ┌─────────────────┐              ┌────────────────────┐
   │ consultar_pedido│              │  retriever (k=4)   │
   │ (PEDIDOS_DICT)  │              │  ChromaDB          │
   └────────┬────────┘              └─────────┬──────────┘
            │                                  │
            ▼                                  ▼
   ┌─────────────────┐              ┌────────────────────┐
   │ prompt_pedido   │              │ prompt_general     │
   │ (formato fijo)  │              │ (anti-alucinación) │
   └────────┬────────┘              └─────────┬──────────┘
            │                                  │
            └─────────────┬────────────────────┘
                          ▼
                   ┌────────────────┐
                   │ llama3.2:3b    │
                   │ (Ollama local) │
                   └────────┬───────┘
                            ▼
                   respuesta + fuentes
```

Dos rutas, una sola función pública (`responder`). El **fallback** "No tengo información sobre eso en los documentos de EcoMarket" no es una rama de código separada: vive en `prompts/prompt_general.txt` como instrucción al modelo, y se activa naturalmente cuando los chunks recuperados no cubren la pregunta.

---

## Cómo verificar que funciona

Ejecutar el notebook completo (`Run All`) desde la raíz del proyecto y observar los resultados de la sección 9:

| # | Pregunta | Ruta esperada | Comportamiento esperado |
|---|---|---|---|
| 1 | `¿Cuál es el estado de mi pedido 1003?` | `tool_pedidos` | Reporta estado "Retrasado", fecha de entrega del JSON, ofrece disculpa. **No** consulta el vectorstore. |
| 2 | `¿Puedo devolver un cepillo de dientes de bambú?` | `rag` | Recupera el producto del catálogo (P001, no devolvible) y/o la política, concluye que la devolución no aplica y explica por qué. |
| 3 | `¿Tienen botellas térmicas y cuánto cuestan?` | `rag` | Recupera P002 del catálogo, responde con el precio real (`$85.000`), no inventa otros modelos. |
| 4 | `¿Cuál es la capital de Francia?` | `rag` | Fuera de dominio: ningún chunk relevante, dispara el fallback "No tengo información…". |

Cada respuesta imprime sus fuentes, lo que permite auditar de qué documento salió la información y refuerza la mitigación del riesgo de alucinaciones identificado en el Taller 1.

---

## Limitaciones y suposiciones

### Limitaciones del prototipo

- **Router por regex.** Cualquier número de 4 dígitos en una pregunta es interpretado como posible tracking. Si un cliente pregunta por un código postal, el router lo intenta primero como pedido. La degradación es benigna (cae al RAG si el ID no existe), pero un router maduro usaría clasificación de intención.
- **Sin filtrado por metadata.** El retriever hace top-k global sobre los tres documentos. Para producción se agregaría `search_kwargs={"filter": {"tipo": "producto"}}` para restringir consultas de catálogo. La metadata ya se preserva por chunk, así que la mejora es de una línea.
- **Modelo pequeño.** `llama3.2:3b` es eficiente pero ocasionalmente no sigue el formato exacto del prompt (límite de líneas, frase exacta del fallback). Un modelo de 7-8B parámetros reduce esa varianza.
- **Sin evaluación cuantitativa.** No se mide hit rate ni MRR del retrieval. La mejora natural sería montar un set de preguntas etiquetadas con la respuesta esperada.
- **Sin UI.** El sistema corre dentro del notebook. Una integración con Gradio o FastAPI sería trivial sobre la función `responder` pero no aporta a los criterios evaluados de la rúbrica.

### Suposiciones de ejecución

- **Hardware**: Windows / Linux / macOS con ≥8 GB de RAM. GPU opcional (acelera la generación de embeddings; el LLM `llama3.2:3b` corre cómodo en CPU).
- **Conectividad**: requerida solo en la primera ejecución, para descargar `llama3.2:3b` (~2 GB) y `intfloat/multilingual-e5-large` (~1.3 GB). Después el sistema funciona 100% local.
- **Datos**: `pedidos.json` se asume sincronizado al momento de la consulta. En producción la `consultar_pedido` consultaría una API o base relacional; el contrato de la función no cambia.
- **Idioma**: las consultas y los documentos están en español. Otros idiomas funcionan (E5 es multilingüe) pero no se han validado.
- **Versiones de librerías**: las versiones fijadas en `requirements.txt` (LangChain 0.3, langchain-chroma 0.1, langchain-ollama 0.2) son las probadas. Versiones mayores podrían requerir ajustes en imports.
