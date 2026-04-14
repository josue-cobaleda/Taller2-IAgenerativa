# 🧠 Taller 1 - IA Generativa (EcoMarket)

Este repositorio contiene el desarrollo del **Taller Práctico #1** de la asignatura de Inteligencia Artificial Generativa, enfocado en el diseño de una solución basada en **LLMs y arquitectura RAG (Retrieval-Augmented Generation)** para mejorar el servicio al cliente de una empresa de e-commerce ficticia llamada **EcoMarket**.

---

## 🎯 Objetivo del proyecto

Diseñar y proponer una solución basada en IA generativa que permita:

- Reducir los tiempos de respuesta en atención al cliente  
- Automatizar consultas repetitivas (≈80%)  
- Mejorar la experiencia del usuario  
- Mantener precisión mediante integración con datos reales  

---

## 🧩 Estructura del repositorio
Taller1-IAGenerativa/
│
├── README.md
├── fase1.md # Selección y justificación del modelo
├── fase2.md # Análisis de riesgos éticos (modelo Huang)
├── fase3.md # Ingeniería de prompts y prototipo
│
├── data/
│ ├── pedidos.json # Base de datos simulada de pedidos
│ └── politicas.txt # Políticas de devolución
│
├── prompts/
│ ├── prompt_pedido.txt
│ └── prompt_devolucion.txt
│
├── embeddings.py # Generación de embeddings
├── rag.py # Recuperación de contexto (mini RAG)
├── app.py # Aplicación principal
└── requirements.txt

---

## 🏗️ Arquitectura de la solución

La solución propuesta sigue un enfoque híbrido basado en RAG.


### Componentes clave:

- **LLM (LLaMA 3 vía Ollama)**: generación de respuestas
- **RAG**: recuperación de información relevante
- **Embeddings**: búsqueda semántica en datos
- **Datos estructurados**: pedidos y políticas
- **Prompt engineering**: control del comportamiento del modelo

---

## ⚙️ Tecnologías utilizadas

- Python 3
- Ollama (LLM local)
- Sentence Transformers (embeddings)
- NumPy (similitud vectorial)
- JSON / TXT (fuentes de datos)
- Prompt Engineering

---

## 🚀 Cómo ejecutar el proyecto

### 1. Instalar dependencias

```bash
pip install -r requirements.txt

python app.py

Cliente: ¿Dónde está mi pedido 1003?

Respuesta esperada:
Estado del pedido
Fecha de entrega
Explicación clara y empática

---

🧠 Enfoque de Prompt Engineering

Se aplicaron técnicas avanzadas de prompting:

Role Prompting
Context Injection (RAG)
Instruction Prompting
Output Formatting
Constraint Prompting
Conditional Logic Prompting
Tone Control

Esto permitió:

Reducir alucinaciones
Mejorar precisión
Controlar estructura de respuestas
Alinear la IA con necesidades empresariales

---

⚖️ Consideraciones éticas

Se realizó un análisis basado en el modelo de Huang, considerando:

Seguridad (alucinaciones, prompt injection)
Privacidad de datos
Sesgo algorítmico
Transparencia
Impacto laboral
Sostenibilidad

Se proponen mecanismos de mitigación como:

RAG con fuentes controladas
Escalamiento a humanos
Minimización de datos
Auditoría de respuestas

---

💡 Resultados clave

La solución permite:

Automatizar consultas frecuentes
Mejorar tiempos de respuesta (de horas a segundos)
Reducir carga operativa
Mantener precisión mediante datos reales
Escalar sin aumentar proporcionalmente el equipo


