# Taller 2 — IA Generativa (EcoMarket + RAG)

Autores:

- Farid Sandoval
- Ivan Morán
- Josué Cobaleda

Este repositorio contiene el desarrollo del **Taller Práctico #2** de la asignatura de Inteligencia Artificial Generativa. Extiende el prototipo del Taller 1 incorporando un **sistema RAG (Retrieval-Augmented Generation)** para que el asistente de atención al cliente de **EcoMarket** pueda responder a cualquier tipo de consulta, o reconocer explícitamente cuándo no tiene información para hacerlo.

---

## Objetivo

Sobre el modelo del Taller 1 (LLaMA 3 vía Ollama + prompts curados):

- Conectar el LLM a una **base de conocimiento vectorial** con los documentos internos de EcoMarket (políticas, catálogo, FAQ).
- Reducir alucinaciones inyectando contexto recuperado en cada respuesta.
- Manejar consultas fuera de dominio con un **fallback explícito** ("No tengo información sobre eso…") en lugar de inventar.
- Mantener la **consulta directa por ID** para datos transaccionales volátiles (pedidos), separándola del retrieval semántico.

---

## Estructura del repositorio

```
Taller2-IAgenerativa/
│
├── README.md
├── fase1-componentes.md            # Selección y justificación de componentes (embeddings + vectorstore)
├── fase2-base conocimiento.md      # Identificación de documentos y estrategia de chunking
├── fase3-integracion.md            # Integración y ejecución del código
├── Model-ollama-langchain.ipynb    # Implementación end-to-end del sistema RAG
├── requirements.txt
│
├── data/
│   ├── politicas_devoluciones.md   # Reglas de negocio (indexado en RAG)
│   ├── catalogo_productos.json     # 15 productos con metadata (indexado en RAG)
│   ├── faq.json                    # 15 pares Q&A curados (indexado en RAG)
│   └── pedidos.json                # Pedidos simulados (NO indexado, consulta directa)
│
└── prompts/
    ├── prompt_pedido.txt           # Prompt usado por la herramienta directa de pedidos
    ├── prompt_general.txt          # Prompt anti-alucinación usado por la cadena RAG
    └── prompt_devolucion.txt       # Conservado como referencia del Taller 1
```

---

## Arquitectura

| Componente | Elección | Justificación detallada |
|---|---|---|
| **LLM** | Ollama + `llama3.2:3b` (local) | [fase1-componentes.md](fase1-componentes.md) |
| **Embeddings** | `intfloat/multilingual-e5-large` (HuggingFace, 1024 dims) | [fase1-componentes.md](fase1-componentes.md) |
| **Vector store** | ChromaDB persistente local (`./chroma_db_ecomarket/`) | [fase1-componentes.md](fase1-componentes.md) |
| **Orquestación** | LangChain (loaders, splitters, retriever, prompts) | [fase1-componentes.md](fase1-componentes.md) |
| **Chunking** | Recursivo (500/50) para Markdown · 1 registro = 1 chunk para JSON | [fase2-base conocimiento.md](fase2-base%20conocimiento.md) |

Flujo runtime: la pregunta del cliente pasa por un router simple que detecta si trae un tracking number; si lo trae, invoca una herramienta determinista sobre `pedidos.json`. En cualquier otro caso, se ejecuta una cadena RAG única que recupera los 4 chunks más similares de la base unificada y los inyecta a un prompt anti-alucinación. Todo el flujo está documentado y diagramado en [fase3-integracion.md](fase3-integracion.md).

---

## Cómo ejecutar el proyecto

### Requisitos previos

- **Python 3.10+** con un entorno virtual activo como kernel del notebook.
- **Ollama** instalado (descargar desde https://ollama.com/download).
- Ejecutar el notebook **desde la raíz del proyecto** (donde viven `data/` y `prompts/`).

### 1. Clonar el repositorio

```bash
git clone https://github.com/josue-cobaleda/Taller2-IAgenerativa.git
cd Taller2-IAgenerativa
```

### 2. Crear y activar un entorno virtual

```bash
python -m venv venv
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate
```

### 3. Instalar dependencias

```bash
pip install -r requirements.txt
```

### 4. Instalar y arrancar Ollama

- Windows: descargar e instalar desde https://ollama.com/download/windows. Tras la instalación, Ollama corre como servicio en `http://localhost:11434`. Si no está activo, abrir una terminal y ejecutar `ollama serve`.
- macOS / Linux: `curl -fsSL https://ollama.com/install.sh | sh`.
- Verificar: `ollama --version`.

### 5. Abrir y ejecutar el notebook

Abrir `Model-ollama-langchain.ipynb` en VS Code o Jupyter Lab con el kernel del venv activo, y ejecutar **Run All**.

### Primera ejecución

El notebook descargará automáticamente:

- `llama3.2:3b` (~2 GB) en Ollama.
- `intfloat/multilingual-e5-large` (~1.3 GB) en `~/.cache/huggingface`.
- Construirá el índice vectorial en `./chroma_db_ecomarket/`.

Las corridas siguientes leen todo desde caché y desde disco, sin re-descargar ni re-indexar.

### Pruebas incluidas (sección 9 del notebook)

| Pregunta | Ruta esperada |
|---|---|
| `¿Cuál es el estado de mi pedido 1003?` | Herramienta directa de pedidos → "Retrasado" |
| `¿Puedo devolver un cepillo de dientes de bambú?` | RAG → no devolvible (recupera del catálogo y la política) |
| `¿Tienen botellas térmicas y cuánto cuestan?` | RAG → precio real del catálogo |
| `¿Cuál es la capital de Francia?` | RAG → fallback "No tengo información…" |

---

## Resultados clave

- **Coherencia con el diseño**: la implementación materializa exactamente las decisiones de las fases 1 y 2, sin atajos.
- **Cobertura completa del enunciado**: el asistente responde a cualquier consulta o reconoce explícitamente cuándo no puede.
- **Trazabilidad**: cada respuesta incluye sus fuentes, lo que facilita auditoría y mitiga el riesgo ético de alucinaciones.
- **Costo marginal cero** y **privacidad total**: todo corre en local (LLM, embeddings, vector store).
