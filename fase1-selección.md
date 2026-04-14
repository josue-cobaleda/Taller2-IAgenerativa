# Fase 1. Selección y justificación del modelo de IA

## 1. Modelo propuesto

Para el caso de **EcoMarket**, se propone una **solución híbrida basada en una LLM (ej.: GPT-4) con arquitectura Transformer, integrado con RAG (Retrieval-Augmented Generation) y conectado a las fuentes de datos internas de la empresa**.

La solución combinaría tres elementos principales:

1. **Un LLM** para comprender las preguntas de los clientes y generar respuestas naturales, claras y útiles.
2. **Un sistema RAG** para recuperar información actualizada desde las bases de datos y documentos de EcoMarket antes de responder.
3. **Un mecanismo de escalar casos complejos a humanos** para aquellos casos que requieran empatía, criterio o atención especializada.

Esta propuesta responde mejor al problema del caso, porque EcoMarket no solo necesita generar texto, sino **responder con precisión usando información del negocio en tiempo real**, especialmente en temas como estado de pedidos, devoluciones y características de productos.

## 2. ¿Por qué este modelo y no otro?

Se eligió una solución híbrida porque EcoMarket tiene dos necesidades distintas al mismo tiempo:
- **Precisión** en consultas transaccionales, como el estado del pedido o si un producto puede devolverse.
- **Fluidez conversacional** para ofrecer respuestas comprensibles, amables y coherentes con una buena experiencia de servicio.

Un **LLM de propósito general sin acceso a información interna** no sería suficiente, porque aunque podría responder de forma fluida, también podría inventar datos o dar información desactualizada. Esto sería riesgoso en un entorno de comercio electrónico, donde el cliente espera respuestas exactas sobre envíos, devoluciones y productos.

Por otro lado, un **modelo pequeño afinado únicamente con datos de la empresa** tampoco sería la mejor opción inicial. Aunque en algunos escenarios puede reducir costos, tendría limitaciones:
- menor flexibilidad ante preguntas variadas;
- necesidad de volver a ajustarlo cuando cambien políticas, productos o estados de pedidos;
- mayor esfuerzo de mantenimiento.

En cambio, una arquitectura con **LLM + RAG + escalamiento humano** permite aprovechar lo mejor de cada enfoque:
- el **LLM** aporta comprensión del lenguaje natural y buena redacción;
- el **RAG** aporta contexto actualizado y confiable;
- el **agente humano** atiende los casos complejos o sensibles.

## 3. Arquitectura propuesta

La arquitectura propuesta sería la siguiente:

<img width="1536" height="1024" alt="ChatGPT Image 13 abr 2026, 07_58_32 p m" src="https://github.com/user-attachments/assets/318587e8-f79e-42b5-9757-8f82f1a4c4a5" />

Utilizaríamos un LLM como base de generación y lo integraríamos con una base de datos vectorial que contenga la información clave de EcoMarket (por ejemplo, estados de pedidos recientes, catálogo de productos, políticas de envío y devolución). Al procesar una consulta del cliente, se convertiría la pregunta en un vector de consulta y se buscarían documentos relevantes en la base vectorial (realizando una búsqueda semántica). Los fragmentos de información recuperados (por ejemplo, “Pedido 12345: Enviado, entrega estimada 15/04/2026”) se adjuntarían al prompt que se envía al LLM. De este modo, el modelo generativo produce la respuesta combinando su conocimiento pre-entrenado con los datos actualizados extraídos de la base de EcoMarket
. La ingesta inicial de documentos implica procesar los registros empresariales (por ejemplo, convertir el historial de órdenes y productos en incrustaciones vectoriales usando un modelo como Amazon Titan Embedding) y almacenarlos en una base de datos vectorial (p.ej., LanceDB o Pinecone).

### Componentes principales

### 3.1. LLM basado en Transformer
El núcleo de la solución sería un modelo de lenguaje grande basado en la arquitectura Transformer. En la práctica, un modelo de propósito general (GPT-4 o similar) manejado vía API, o un LLM open-source afinado ligeramente, serviría de núcleo. Este componente sería responsable de:
- interpretar la intención del cliente;
- comprender preguntas escritas en lenguaje natural;
- generar respuestas claras, útiles y con tono amable;
- estructurar la información de forma conversacional.


### 3.2. Módulo RAG
El módulo RAG permitiría recuperar información específica desde las fuentes de conocimiento de EcoMarket antes de que el modelo genere una respuesta. Esto evita que el sistema dependa únicamente del conocimiento general del modelo y mejora la precisión.

### 3.3. Fuentes de datos de EcoMarket
La solución debería integrarse con las siguientes fuentes de información:

- base de datos de pedidos y envíos;
- políticas de devoluciones y reembolsos;
- catálogo de productos;
- preguntas frecuentes;
- historial de interacciones del cliente, cuando sea pertinente.

### 3.4. Reglas de negocio y enrutamiento
El sistema también debería contar con reglas para decidir cuándo responder automáticamente y cuándo escalar a un agente humano. Por ejemplo:

- consultas sobre estado del pedido, tiempos de entrega y políticas generales: **respuesta automática**;
- quejas delicadas, reclamos complejos, problemas técnicos o situaciones sensibles: **escalamiento a humano**.

### 3.5. Supervisión humana
La solución no busca reemplazar por completo a los agentes de soporte, sino **reducir la carga operativa del 80% de consultas repetitivas** para que el equipo humano pueda concentrarse en el 20% de casos que requieren más empatía o análisis.

## 4. Justificación según criterios clave

## 4.1. Costo
La solución propuesta permite un mejor equilibrio entre inversión y beneficio:
- automatiza gran parte de las consultas repetitivas;
- reduce la carga operativa del equipo de atención al cliente;
- evita el costo de entrenar o reajustar continuamente un modelo propio;
- permite empezar con un piloto y crecer de forma progresiva.

Aunque usar un LLM tiene costos de infraestructura o consumo por API, estos pueden justificarse por el ahorro en tiempo de respuesta y eficiencia operativa.

## 4.2. Escalabilidad
La arquitectura es altamente escalable porque puede:
- atender múltiples solicitudes al mismo tiempo;
- operar 24/7;
- integrarse en varios canales de atención;
- crecer junto con el volumen de clientes sin depender linealmente de aumentar el número de agentes.

Esto es especialmente importante porque EcoMarket está creciendo rápidamente y ya presenta un cuello de botella en soporte.

## 4.3. Facilidad de integración
La propuesta es realista desde el punto de vista técnico, ya que puede conectarse con sistemas que normalmente existen en una empresa de e-commerce, como:
- plataforma de pedidos;
- CRM;
- base de conocimiento;
- sistema de inventario;
- canales digitales de atención.

Además, herramientas como LangChain o LlamaIndex pueden facilitar la implementación de una solución con RAG, ya que ayudan a conectar modelos generativos con fuentes de información empresariales.

## 4.4. Calidad de la respuesta esperada
La calidad esperada de la respuesta es alta porque la solución combina:
- capacidad conversacional del LLM;
- acceso a información actualizada del negocio;
- control mediante reglas y supervisión humana.

Esto permite mejorar la precisión en consultas sobre pedidos y devoluciones; la consistencia de las respuestas; la rapidez del servicio y  la experiencia del cliente.
