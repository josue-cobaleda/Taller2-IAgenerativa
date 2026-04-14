# Fase 2. Evaluación de fortalezas, limitaciones y riesgos éticos

La solución propuesta para EcoMarket consiste en una arquitectura híbrida basada en un **LLM con arquitectura Transformer**, apoyado por un sistema **RAG** e integrado con las fuentes de datos internas de la empresa. Esta alternativa tiene un alto potencial para mejorar el servicio al cliente, pero también implica limitaciones técnicas y riesgos éticos que deben ser analizados antes de su implementación.

## 1. Fortalezas de la solución propuesta

La arquitectura RAG aporta una significativa precisión y actualidad a las respuestas. Al incluir datos en tiempo real (p.ej. estado real de un pedido), se evitan respuestas obsoletas propias de LLM estáticos. Además, al basarse en información autorizada, RAG reduce las “alucinaciones” (errores no basados en datos reales) típicas de los LLM puros. Esto mejora la confianza del usuario, ya que la salida puede incluir referencias explícitas a fuentes (por ejemplo, “Según nuestro registro, su pedido 12345 fue enviado el…”). También se facilita la personalización: el sistema puede recuperar el historial específico del cliente (compras anteriores, preferencias) para adaptar la respuesta. Por ejemplo, en atención al cliente Esker reporta que RAG permite “generar respuestas basadas en contenido aprobado y actualizado” integrando datos de ERP y CRM, lo cual se traduciría en respuestas más eficientes y satisfactorias (especialmente para el 80% de consultas repetitivas).

### - Reducción del tiempo de respuesta

La principal fortaleza de la solución es su capacidad de responder de manera automática a gran parte de las consultas repetitivas. En el caso de EcoMarket, esto es especialmente relevante porque el problema central consiste en que el equipo de soporte recibe miles de consultas diarias y mantiene un tiempo de respuesta promedio de 24 horas.

### - Disponibilidad continua

Otra fortaleza importante es la posibilidad de ofrecer atención **24/7**. A diferencia de un equipo humano, el sistema puede responder en cualquier momento del día, lo que mejora la experiencia del cliente, especialmente en canales digitales como chat, correo o redes sociales.

### - Manejo eficiente del 80% de consultas repetitivas

La solución está bien alineada con la naturaleza del problema, ya que el caso plantea que aproximadamente el **80% de las consultas son repetitivas**. En este contexto, la IA generativa combinada con RAG puede encargarse de esas solicitudes frecuentes de manera eficiente, dejando al equipo humano más tiempo para atender los casos complejos.

### - Respuestas más consistentes

El sistema puede ayudar a estandarizar la calidad de la atención, evitando variaciones innecesarias entre agentes y garantizando que las respuestas sobre políticas, devoluciones o productos mantengan una línea uniforme.

### - Mejor aprovechamiento del talento humano

Una fortaleza estratégica de esta solución es que no necesariamente reemplaza a los agentes, sino que puede liberar al equipo de tareas repetitivas, los empleados pueden enfocarse en actividades de mayor valor, como resolver reclamos delicados, atender clientes inconformes o gestionar situaciones excepcionales.

### - Escalabilidad operativa

A medida que EcoMarket continúe creciendo, una solución basada en IA generativa puede atender un mayor volumen de consultas sin que el crecimiento del equipo de soporte tenga que aumentar al mismo ritmo. Esto hace que la solución sea atractiva desde una perspectiva operativa y empresarial.

## 2. Limitaciones de la solución propuesta

Implementar RAG implica una compleja canalización técnica. Se requieren recursos para incrustar (vectorizar) todos los documentos empresariales y mantener la base de datos vectorial actualizada. Además, los LLM tienen un límite de tokens; por ello la búsqueda debe ser muy relevante, pues no se puede volcar un documento entero al prompt. Microsoft señala que RAG enfrenta desafíos prácticos: debe comprender consultas vagas o conversacionales y balancear exhaustividad contra rapidez (los usuarios esperan respuestas en segundos). Si la información de la base es incompleta o se desactualiza, el sistema podría devolver respuestas parciales o incorrectas. Por otro lado, el costo computacional puede crecer con el tamaño de la base de datos: más documentos implica más cálculos de similitud vectorial. Finalmente, como toda solución IA, depende de la calidad de los embeddings y del LLM; un mal ajuste en alguno puede degradar el desempeño global.

### - Incapacidad para manejar adecuadamente todos los casos complejos

La principal limitación del sistema es que no puede resolver bien todas las situaciones. Aunque puede responder consultas frecuentes, no sustituye la empatía, el juicio ni la flexibilidad de una persona en casos como:
- quejas sensibles;
- reclamos por experiencias negativas;
- problemas técnicos poco frecuentes;
- clientes frustrados o emocionalmente afectados.

Esto significa que la IA no debe operar como único canal de atención.

### - Dependencia de la calidad de los datos

La calidad de la respuesta dependerá en gran medida de la calidad de las fuentes de datos internas. Si la información de pedidos, devoluciones, productos o preguntas frecuentes está incompleta, desactualizada o mal organizada, el sistema generará respuestas incorrectas o confusas.

En otras palabras, una mala base de conocimiento produce una mala experiencia, incluso si el modelo de IA es bueno.

### - Riesgo de respuestas incorrectas o ambiguas

Aunque el uso de RAG reduce errores, no elimina completamente el problema. El modelo aún puede interpretar mal una consulta, mezclar información o responder con ambigüedad si el contexto recuperado no es suficiente o si la pregunta del usuario es confusa.

### - Necesidad de supervisión y mantenimiento constante

La implementación de una solución de este tipo no es algo que se configure una sola vez y luego funcione perfectamente. Requiere:

- actualización de bases de datos y documentos;
- monitoreo de calidad de respuestas;
- ajustes en prompts y reglas de negocio;
- revisión de casos en los que el sistema falla.

Por tanto, no es una solución mágica ni completamente autónoma.

### - Limitaciones para comprender contexto humano profundo

Aunque el modelo pueda parecer conversacional y natural, no comprende emociones humanas de forma real. Puede simular empatía lingüística, pero no experimentar empatía genuina. Esto limita su capacidad para tratar situaciones delicadas con la sensibilidad adecuada. Tampoco siente miedo de que lo echen por una decisión, ni tampoco tiene sentido de responsabilidad en determinados casos. 

## 3. Matriz de riesgos éticos según Huang

<img width="419" height="371" alt="image" src="https://github.com/user-attachments/assets/e2f7b4ce-1955-4da8-9c10-5a8a7e06e314" />

A continuación se presenta el análisis de riesgos éticos de la solución propuesta para EcoMarket, utilizando como marco la categorización de problemas éticos de IA planteada por Huang. La tabla adapta dichas categorías al contexto de una arquitectura híbrida basada en LLM + RAG para atención al cliente en e-commerce.

| Categoría | Riesgo | Posible mitigación |
|---|---|---|
| **Seguridad (nivel individual)** | El sistema puede generar respuestas incorrectas sobre pedidos, devoluciones o productos, afectando directamente al cliente. También existe riesgo de *prompt injection*, abuso del canal y extracción no autorizada de información interna si el asistente recibe instrucciones maliciosas. | Implementar arquitectura **RAG** con acceso restringido solo a fuentes autorizadas; aplicar validaciones de negocio antes de responder sobre pedidos o devoluciones; usar filtros contra *prompt injection*; limitar acciones sensibles; registro de auditoría; pruebas de seguridad periódicas; escalamiento automático cuando la confianza de la respuesta sea baja. |
| **Privacidad y protección de datos (nivel individual)** | La solución procesará datos personales como nombre, dirección, historial de compras, número de pedido y reclamos. Un mal diseño podría exponer datos de un cliente a otro, almacenar más información de la necesaria o usar datos sensibles en prompts sin control adecuado. | Aplicar principio de **minimización de datos**; anonimizar o seudonimizar cuando sea posible; separar datos transaccionales de datos conversacionales; cifrado en tránsito y reposo; control de acceso por roles; políticas claras de retención; enmascaramiento de datos en logs; revisión legal y de cumplimiento en protección de datos. |
| **Libertad y autonomía (nivel individual)** | Un sistema demasiado automatizado puede limitar la capacidad del cliente para elegir cómo ser atendido, forzándolo a interactuar con la IA aunque necesite soporte humano. También puede inducir decisiones del cliente mediante respuestas persuasivas o incompletas. | Garantizar siempre una opción visible de **hablar con un humano**; no ocultar rutas de escalamiento; diseñar respuestas informativas y no manipulativas; incluir confirmaciones en decisiones relevantes; permitir que el usuario corrija o aclare el contexto fácilmente. |
| **Dignidad humana (nivel individual)** | Si la IA responde de manera fría, estandarizada o inadecuada ante reclamos sensibles, puede hacer sentir al cliente ignorado, tratado como un caso mecánico y no como persona. Esto es especialmente delicado en quejas, pérdidas o frustraciones fuertes. | Diseñar reglas para detectar casos sensibles y escalar; usar lineamientos de tono empático sin simular emociones falsas; entrenar al equipo humano para atender situaciones complejas; medir satisfacción del usuario no solo en velocidad sino en percepción de trato digno. |
| **Equidad y justicia (nivel societal)** | El modelo puede responder mejor a ciertos perfiles de usuarios que a otros, por ejemplo según forma de redactar, nivel educativo, lenguaje más formal o tipo de reclamo. Esto puede traducirse en desigualdad en la calidad del servicio. | Evaluar el sistema con consultas diversas; monitorear desempeño por tipo de cliente, canal y estilo de interacción; establecer criterios uniformes de servicio; revisar sesgos en reglas, prompts y datos históricos; mantener revisión humana de casos sensibles o ambiguos. |
| **Responsabilidad y rendición de cuentas (nivel societal)** | Si una respuesta de la IA causa un perjuicio, puede volverse difuso quién responde: el modelo, el equipo técnico, la empresa o el agente. La falta de trazabilidad dificulta corregir errores y asumir responsabilidad. | Definir gobernanza clara: la **empresa sigue siendo responsable** de la decisión automatizada; mantener trazabilidad de fuentes consultadas, prompts, respuestas y escalaciones; versionar prompts y reglas; asignar responsables funcionales, técnicos y legales del sistema. |
| **Transparencia (nivel societal)** | El cliente puede no saber si habla con una IA o con un humano, ni de dónde sale la respuesta. Esto afecta la confianza y dificulta cuestionar decisiones automatizadas. | Informar explícitamente que el primer nivel de atención es asistido por IA; señalar cuándo una respuesta fue generada con base en políticas o datos del sistema; mostrar explicaciones breves cuando aplique; permitir revisión humana si el cliente no está de acuerdo. |
| **Vigilancia y dataficación (nivel societal)** | La empresa podría empezar a capturar y analizar cada interacción del cliente más allá de lo necesario, convirtiendo la atención en un mecanismo de vigilancia comercial o perfilamiento excesivo. | Limitar la recopilación al propósito de atención; evitar reutilización irrestricta de conversaciones para fines comerciales; gobernanza de datos con finalidad específica; políticas de consentimiento y uso secundario; métricas agregadas en lugar de monitoreo individual innecesario. |
| **Controlabilidad de la IA (nivel societal)** | Si la organización depende demasiado del sistema, puede perder capacidad de intervención humana efectiva. También puede ocurrir que el modelo responda fuera del alcance previsto o que se amplíe su uso sin controles suficientes. | Definir límites funcionales claros; usar guardrails y reglas de negocio; umbrales de confianza para automatización; botón de parada o desactivación operativa; monitoreo continuo; revisión periódica de alcance para evitar deriva funcional. |
| **Democracia y derechos civiles (nivel societal)** | Aunque el caso es empresarial, una mala gestión algorítmica puede afectar derechos del consumidor, por ejemplo negando devoluciones, ocultando opciones legítimas o dificultando reclamaciones de manera sistemática. | Alinear el sistema con principios de protección al consumidor; revisar que la automatización no restrinja derechos ni vías de reclamación; auditorías internas de cumplimiento; mecanismos simples para apelar una respuesta automatizada. |
| **Sustitución laboral (nivel societal)** | La implementación puede ser usada como argumento para reducir personal sin rediseñar el trabajo de forma responsable, generando pérdida de empleo, desmotivación y resistencia interna. | Enfocar la IA como herramienta de **apoyo y redistribución de tareas**; capacitar agentes para supervisión, resolución compleja y mejora del sistema; transición organizacional planificada; indicadores de calidad humana, no solo reducción de costos. |
| **Relación humana (nivel societal)** | Una atención excesivamente automatizada puede deteriorar la relación entre empresa y cliente, haciendo que la interacción sea más eficiente pero menos humana, menos confiable y menos relacional. | Diseñar una experiencia híbrida: automatización para lo repetitivo y presencia humana para lo complejo; definir momentos donde la intervención humana agrega valor; medir no solo eficiencia sino confianza, satisfacción y lealtad. |
| **Recursos naturales (nivel ambiental)** | El despliegue de modelos de IA y su infraestructura de cómputo consume recursos materiales y tecnológicos que no son neutrales, especialmente si se escala sin criterio. | Elegir una arquitectura proporcional al problema; evitar modelos sobredimensionados; reutilizar infraestructura existente cuando sea viable; priorizar eficiencia computacional en vez de complejidad innecesaria. |
| **Energía (nivel ambiental)** | El uso continuo de LLMs, consultas frecuentes y almacenamiento asociado incrementa el consumo energético del servicio al cliente. | Usar modelos y flujos eficientes; limitar longitud de contexto; cachear respuestas frecuentes; optimizar consultas RAG; monitorear consumo por volumen de interacción; elegir proveedores con mejores prácticas energéticas cuando sea posible. |
| **Contaminación ambiental (nivel ambiental)** | El incremento de infraestructura digital y renovación tecnológica puede contribuir indirectamente a huella de carbono y residuos electrónicos. | Adoptar criterios de compra tecnológica responsable; extender vida útil de infraestructura; seleccionar proveedores con políticas ambientales; medir huella digital aproximada del sistema en fases de escalamiento. |
| **Sostenibilidad (nivel ambiental)** | Una solución puede ser técnicamente exitosa pero insostenible si su costo ambiental, energético y operativo crece sin control. | Evaluar sostenibilidad desde el diseño: eficiencia, costo, impacto, utilidad real; implementar por etapas; revisar periódicamente si el valor generado justifica los recursos consumidos; incorporar indicadores técnicos, éticos y ambientales en la gobernanza del sistema. |

