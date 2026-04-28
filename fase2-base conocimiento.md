# Fase 2 — Creación de la base de conocimiento

## Principio rector

> La calidad del retrieval nunca es mejor que la calidad de los documentos que lo alimentan.

Un sistema RAG perfectamente configurado que apunta a documentos mal estructurados o mal segmentados producirá respuestas pobres. Por eso esta fase es donde se resuelve la mayor parte del rendimiento real del sistema, antes de escribir una sola línea de código de orquestación.

Se definen tres decisiones:

1. **Qué documentos componen la base de conocimiento.**
2. **Cómo se segmentan** (chunking) para que el retrieval los encuentre bien.
3. **Cómo se indexan** (el pipeline de ingesta).

---

## 1. Identificación de documentos

Para que el agente de atención al cliente de EcoMarket pueda resolver las consultas más frecuentes sin alucinar, la base de conocimiento debe cubrir tres dominios informativos distintos:

| Dominio | Documento | Formato | Origen | Uso en RAG |
|---|---|---|---|---|
| **Políticas y procedimientos** | `politicas_devoluciones.md` | Markdown | Legal / operaciones | Sí — indexado |
| **Catálogo de productos** | `catalogo_productos.json` | JSON estructurado | ERP / sistema de inventario | Sí — indexado |
| **Preguntas frecuentes** | `faq.json` | JSON estructurado | Soporte al cliente | Sí — indexado |
| *(complemento)* Estado de pedidos | `pedidos.json` | JSON transaccional | Sistema de órdenes | **No indexado — consulta directa** |

### Justificación de cada elección

**Políticas de devoluciones** (`politicas_devoluciones.md`)
Contiene las reglas de negocio que determinan si un producto es retornable, el plazo, las condiciones y el proceso. Es texto semi-estructurado con excepciones y cláusulas — el caso clásico donde el retrieval semántico aporta valor, porque el cliente nunca pregunta literalmente *"¿cuál es la política sobre productos perecederos?"*, sino *"¿puedo devolver unas frutas que llegaron feas?"*. El embedding captura esa equivalencia.

**Catálogo de productos** (`catalogo_productos.json`)
Información estructurada sobre cada producto: id, nombre, descripción, precio, categoría, material, si admite devolución, stock. Permite al modelo responder preguntas de tipo *"¿tienen cepillos de dientes biodegradables?"*, *"¿cuánto cuesta el set de bolsas reutilizables?"* o *"¿se puede devolver el detergente?"* sin inventar productos ni precios.

**FAQ** (`faq.json`)
Pares pregunta–respuesta curados por el equipo de soporte. Es la fuente de mayor precisión porque ya están redactados con el tono oficial y cubren el 80% de consultas repetitivas del caso (según el enunciado del Taller 1). El embedding de las preguntas funciona especialmente bien para el retrieval porque la forma superficial de una consulta del cliente se parece mucho a la forma superficial de las preguntas de la FAQ.

### Por qué los pedidos NO se indexan en el vector store

`pedidos.json` contiene información transaccional volátil (estado, fecha, tracking). Dos razones para excluirlo del RAG:

- **No es búsqueda semántica.** La consulta *"¿dónde está mi pedido 1003?"* no se resuelve por similitud de significado, sino por **búsqueda exacta por ID**. Usar embeddings para esto es ineficiente y propenso a errores (dos IDs numéricos pueden tener embeddings parecidos sin significar lo mismo).
- **Los datos cambian constantemente.** Reindexar embeddings cada vez que un pedido cambia de estado sería costoso y lento. Es mejor consultar el JSON (o la base relacional que lo represente) como **herramienta separada**, invocada por el agente cuando detecte que la consulta menciona un número de tracking.

Este patrón — **RAG para conocimiento estable + consulta directa a fuente para datos volátiles** — es el diseño estándar de un agente de atención al cliente maduro.

---

## 2. Estrategia de segmentación (chunking)

### El problema

Los LLMs tienen ventana de contexto limitada y, más importante, los embeddings pierden calidad cuando representan textos muy largos (el significado se "diluye"). Pero si los chunks son demasiado cortos, se pierde contexto y el modelo recibe fragmentos inútiles. La segmentación resuelve este trade-off.

### Estrategias consideradas

| Estrategia | Descripción | Ventaja | Desventaja |
|---|---|---|---|
| **Tamaño fijo** (carácter o token) | Corta cada N caracteres sin mirar el contenido | Simple, predecible | Rompe oraciones, párrafos y listas a la mitad |
| **Por párrafos** | Corta en saltos de línea dobles | Respeta unidades de significado | Párrafos muy largos quedan iguales, muy cortos pierden contexto |
| **Recursivo (jerárquico)** | Intenta cortar por el separador más grande posible (`\n\n`, luego `\n`, luego `. `, luego ` `) hasta caber en el tamaño objetivo | Respeta jerarquía semántica natural | Ligeramente más costoso de configurar |
| **Semántico** | Agrupa oraciones por similitud de embedding | Máxima coherencia temática | Costo computacional alto, dependencia del embedder |
| **Por documento completo** | Un documento = un chunk | Conserva todo el contexto | Solo viable para docs muy cortos; inútil para retrieval fino |

### Decisión

Se usan **dos estrategias combinadas según el tipo de documento**, porque aplicar una sola estrategia ciega a todo sería subóptimo:

#### (a) Para texto no estructurado — `politicas_devoluciones.md`

**Estrategia: `RecursiveCharacterTextSplitter` de LangChain.**

Parámetros:
- `chunk_size = 500` caracteres
- `chunk_overlap = 50` caracteres
- `separators = ["\n\n", "\n", ". ", " ", ""]`

**Justificación:**

- **Por qué recursivo y no tamaño fijo.** Las políticas tienen estructura jerárquica natural: secciones (`##`), párrafos, oraciones. El splitter recursivo respeta esa jerarquía intentando primero cortar por párrafos y solo bajando a oraciones o palabras si un párrafo es muy largo. Esto produce chunks semánticamente coherentes — una condición de devolución se mantiene junta, no cortada por la mitad.
- **Por qué 500 caracteres.** Las políticas de EcoMarket son relativamente compactas; 500 caracteres (≈ 80–100 palabras) capturan una cláusula completa sin diluir la señal semántica del embedding. Chunks mucho más grandes (ej. 2000) reducen precisión de retrieval porque el embedding "promedia" varias ideas. Chunks mucho más pequeños (ej. 100) fragmentan las reglas de negocio y obligan a recuperar más top-k, aumentando ruido.
- **Por qué overlap de 50.** Si una condición crítica cae justo en el borde entre dos chunks (ej. "los productos perecederos NO son devolvibles" con la negación aislada), el modelo podría recibir solo la mitad. 50 caracteres de solape (≈ 10% del chunk) garantizan que la transición entre fragmentos preserve el contexto inmediato sin inflar demasiado el número total de chunks.

#### (b) Para datos estructurados — `catalogo_productos.json` y `faq.json`

**Estrategia: un registro = un chunk atómico.**

Implementación: cada entrada del JSON se convierte en un string legible (tipo *"Producto: Cepillo de bambú. Categoría: higiene. Precio: $15.000. Material: bambú. Admite devolución: no (producto de higiene personal)"*) y ese string se indexa como un único documento.

**Justificación:**

- **La unidad de significado ya existe.** Un producto o una pregunta-respuesta son unidades atómicas de información. Partirlas por caracteres no solo es innecesario, sino que degrada el retrieval (un chunk con solo el precio sin el nombre del producto es inútil).
- **Cada registro tiene metadata valiosa.** Se aprovechan los campos estructurados (`categoria`, `devolvible`, `producto_id`) como **metadata del vector** en ChromaDB, lo que permite filtrado híbrido en el retrieval (ej. recuperar solo productos de la categoría "hogar").
- **Tamaño controlado por diseño.** Una entrada de catálogo o FAQ rara vez excede los 500–800 caracteres, por lo que el chunk atómico cae naturalmente dentro del rango óptimo de los embeddings.

### Tabla resumen de chunking

| Documento | Estrategia | Tamaño objetivo | Overlap | Metadata preservada |
|---|---|---|---|---|
| `politicas_devoluciones.md` | Recursive Character | 500 chars | 50 chars | `source=politicas`, `seccion` |
| `catalogo_productos.json` | 1 registro = 1 chunk | variable (~200–400 chars) | n/a | `source=catalogo`, `producto_id`, `categoria`, `devolvible` |
| `faq.json` | 1 par Q&A = 1 chunk | variable (~150–300 chars) | n/a | `source=faq`, `tema` |

---

## 3. Proceso de indexación

La indexación es un proceso **offline** (se ejecuta una vez al preparar el sistema, y se re-ejecuta solo cuando los documentos cambian). Separarlo claramente de la consulta online es una decisión arquitectónica importante: no se quiere pagar el costo de re-embedding en cada pregunta del usuario.

### Pipeline en 5 pasos

```
1. LOAD          2. SPLIT           3. EMBED           4. STORE           5. PERSIST
┌──────────┐    ┌──────────┐       ┌──────────┐       ┌──────────┐      ┌──────────┐
│ Loaders  │───▶│ Splitters│──────▶│ e5-large │──────▶│ ChromaDB │─────▶│  ./disk  │
│ por tipo │    │ por tipo │       │  (HF)    │       │ in-memory│      │  .chroma │
└──────────┘    └──────────┘       └──────────┘       └──────────┘      └──────────┘
```

### Detalle de cada paso

**1. Load — Carga de documentos**
Se usan loaders específicos por formato:
- `TextLoader` o `UnstructuredMarkdownLoader` para `politicas_devoluciones.md`.
- `JSONLoader` con `jq_schema` para `catalogo_productos.json` y `faq.json`, de forma que cada registro se extraiga como un `Document` independiente con su metadata.

Esto convierte los archivos crudos en objetos `Document` de LangChain, que es la abstracción común que el resto del pipeline entenderá.

**2. Split — Segmentación**
Aplicar la estrategia de la sección 2:
- Markdown → `RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)`
- JSON → ya viene atomizado por el loader, no se re-segmenta.

Resultado: lista unificada de `Document` listos para embeber.

**3. Embed — Generación de vectores**
Cada chunk se pasa por `HuggingFaceEmbeddings(model_name="intfloat/multilingual-e5-large")`. El modelo produce un vector de 1024 dimensiones por chunk. Este paso es el más costoso computacionalmente y la razón principal por la que la indexación se hace offline.

**Nota técnica específica de E5:** este modelo espera el prefijo `"passage: "` al indexar documentos y `"query: "` al buscar. LangChain lo maneja automáticamente, pero es importante confirmarlo al configurar.

**4. Store — Almacenamiento en la base vectorial**
Los vectores, el texto original y la metadata se cargan en ChromaDB usando `Chroma.from_documents(...)`. Chroma construye internamente el índice para búsqueda eficiente por similitud coseno.

**5. Persist — Persistencia a disco**
Con `persist_directory="./chroma_db"` los vectores quedan guardados en disco, de modo que las consultas posteriores simplemente cargan el índice (`Chroma(persist_directory=...)`) sin reprocesar nada. Esto convierte la indexación en un costo de **una vez** en lugar de un costo por sesión.

### Contrato del pipeline

Al finalizar el proceso de indexación, el sistema debe cumplir:

- [ ] Los tres tipos de documentos quedan representados en la misma colección vectorial.
- [ ] Cada chunk conserva metadata que permite rastrear su origen (`source`) y filtrar búsquedas.
- [ ] La base vectorial persiste en disco y puede recargarse sin reprocesar documentos.
- [ ] La adición de un nuevo producto o una nueva política debe poder hacerse incrementalmente (`vectorstore.add_documents(...)`) sin regenerar todo el índice.

---

## Resumen

| Pregunta | Respuesta |
|---|---|
| ¿Qué documentos? | Políticas (MD), Catálogo (JSON), FAQ (JSON) → indexados. Pedidos (JSON) → consulta directa. |
| ¿Cómo se cortan? | Recursive Character para texto libre (500/50); un registro = un chunk para JSON estructurado. |
| ¿Cómo se indexan? | Pipeline offline Load → Split → Embed → Store → Persist, con ChromaDB persistente. |
| ¿Cómo se enriquecen? | Metadata por chunk (`source`, `categoria`, `devolvible`, etc.) para búsqueda híbrida. |
| ¿Cómo se actualiza? | Indexación incremental (`add_documents`), no reindexación completa. |

El resultado es una base de conocimiento donde la **estrategia de segmentación se adapta a la naturaleza del documento** — no una receta única aplicada ciegamente — y donde la separación entre **conocimiento estable (RAG)** y **datos volátiles (consulta directa)** está explícita desde el diseño.