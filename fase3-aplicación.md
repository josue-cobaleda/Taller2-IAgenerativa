# Fase 3. Aplicación de ingeniería de prompts

En esta fase se diseñaron prompts estructurados para resolver dos casos principales:

- Consulta de estado de pedido
- Solicitud de devolución

Se utilizó un enfoque tipo RAG, donde el modelo no responde libremente, sino con base en datos proporcionados.

## Proceso de diseño de prompts

Los prompts finales que se presentan en esta fase no surgieron en un solo intento. Llegamos a ellos mediante un proceso iterativo de ensayo y error: partimos de instrucciones básicas y simples, probamos las respuestas del modelo, identificamos dónde fallaba (alucinaciones, respuestas genéricas, falta de empatía) y fuimos refinando progresivamente las instrucciones hasta lograr resultados consistentes y alineados con las necesidades de EcoMarket. Este proceso es precisamente la esencia de la ingeniería de prompts: iterar hasta encontrar la combinación óptima de rol, contexto, restricciones y formato.

---

## a. Prompt de estado de pedido

Se diseñó un prompt que:

- Define rol (agente de servicio)
- Incluye contexto (base de pedidos)
- Limita al modelo (no inventar información)
- Define estructura de respuesta

Esto permite reducir alucinaciones y mejorar precisión.

---

## b. Prompt de devolución

Se diseñó un prompt que:

- Usa políticas reales como contexto
- Obliga a diferenciar productos permitidos y no permitidos
- Exige empatía en la respuesta

---


Así el sistema logra:

- Respuestas claras y consistentes
- Reducción de errores
- Mayor control del comportamiento del modelo

---


## Técnicas de Prompt Engineering utilizadas

Para mejorar la calidad, precisión y control de las respuestas del modelo, se aplicaron las siguientes técnicas de ingeniería de prompts:

### 1. Role Prompting (definición de rol)
Se define explícitamente el rol del modelo como “agente experto de servicio al cliente de EcoMarket”.  
Esto permite:
- Alinear el tono
- Reducir ambigüedad
- Mejorar consistencia en respuestas

---

### 2. Context Injection (inyección de contexto)
Se proporciona información estructurada (pedidos o políticas) dentro del prompt mediante variables como `{data}` y `{politica}`.

Esto es clave en RAG porque:
- Reduce alucinaciones
- Obliga al modelo a responder con datos reales
- Mejora precisión en escenarios empresariales

---

### 3. Instruction Prompting (instrucciones explícitas)
Se incluyen reglas claras como:
- “NO inventes información”
- “Responde solo con base en los datos”
- “Máximo X líneas”

Esto permite:
- Controlar el comportamiento del modelo
- Reducir respuestas largas o irrelevantes
- Mejorar la calidad de salida

---

### 4. Output Formatting (estructura de salida)
Se define un formato esperado de respuesta:

Ejemplo:
- Estado
- Fecha
- Explicación

Esto ayuda a:
- Estandarizar respuestas
- Facilitar integración en sistemas reales
- Mejorar experiencia del usuario

---

### 5. Constraint Prompting (restricciones)
Se agregan límites como:
- Máximo número de líneas
- Uso exclusivo del contexto
- Prohibición de inventar información

Esto es clave para:
- Reducir alucinaciones
- Aumentar confiabilidad en entornos empresariales

---

### 6. Conditional Logic Prompting
Se incluyen condiciones dentro del prompt:

Ejemplo:
- “Si está retrasado → pedir disculpas”
- “Si no aplica devolución → explicar razón”

Esto permite:
- Simular lógica de negocio
- Hacer respuestas más inteligentes sin código adicional

---

### 7. Tone Control (control de tono)
Se especifica explícitamente:
- “amable”
- “profesional”
- “empático”

Esto es importante en atención al cliente porque:
- Mejora la percepción del usuario
- Alinea la IA con la marca

---
