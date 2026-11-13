# Informe técnico del agente LLM con RAG y herramientas

**Proyecto:** Copiloto de movilidad de Medellín  
**Responsables de la entrega y revisión documental:** Isabella Cadavid Posada y Wendy  
**Componente:** Agente conversacional y recuperación documental

## 1. Objetivo

El agente permite consultar rutas sobre la red vial del proyecto y responder preguntas relacionadas con los documentos oficiales del Metro de Medellín.

Combina un modelo de lenguaje con herramientas que realizan cálculos y consultas sobre datos locales. Para las preguntas documentales utiliza recuperación aumentada por generación —RAG—, incorporando fragmentos del corpus como contexto para elaborar respuestas con fuentes.

Este informe describe el diseño del agente, las herramientas disponibles, la recuperación documental, la evaluación registrada y sus limitaciones.

## 2. Diseño del agente

El agente se implementa en `src/agent/agent.py`. Su función pública recibe una pregunta y el historial de conversación:

```python
ask(message, history)
```

La ejecución sigue un ciclo de llamadas a herramientas:

1. El sistema proporciona al modelo las instrucciones, el historial y la pregunta.
2. El modelo decide si puede responder o necesita utilizar una herramienta.
3. Si solicita una herramienta, el sistema valida sus argumentos y la ejecuta.
4. El resultado se devuelve al modelo como una observación.
5. El ciclo continúa hasta obtener una respuesta final o alcanzar el límite configurado.

El máximo es de **cinco llamadas al modelo por pregunta**. Cuando se alcanza ese límite, el agente comunica la situación y finaliza.

### Estructura de respuesta

| Campo | Contenido |
|---|---|
| `answer` | Respuesta destinada al usuario |
| `sources` | Fuentes citadas, con título, URL, página y fragmento |
| `tool_trace` | Registro de herramientas, argumentos y resultados |
| `error` | Información de error para su tratamiento en la interfaz |

La traza permite inspeccionar cómo se obtuvo una respuesta y distinguir los resultados calculados por las herramientas del texto generado por el modelo.

## 3. Configuración del proveedor

La comunicación con el proveedor se encapsula en `src/agent/llm.py`.

El cliente utiliza una API compatible con `chat completions` y llamadas a herramientas. La configuración se obtiene de las siguientes variables:

```dotenv
LLM_API_KEY=
LLM_BASE_URL=
LLM_MODEL=
```

Los valores reales se guardan en `.env`, que permanece fuera del control de versiones. `.env.example` contiene únicamente la estructura de configuración, sin claves reales.

La evaluación registrada utilizó:

| Parámetro | Valor |
|---|---|
| Proveedor | Groq |
| Modelo | `openai/gpt-oss-120b` |
| Backend de recuperación | `hashing` |

Las pruebas automáticas del agente utilizan un cliente simulado. Permiten comprobar el comportamiento del código sin conexión ni credenciales, pero no sustituyen las pruebas con un proveedor real.

## 4. Herramientas disponibles

El agente dispone de cuatro herramientas:

| Herramienta | Función |
|---|---|
| `plan_route` | Calcula una ruta mediante A* o Dijkstra |
| `get_route_metrics` | Ejecuta y compara ambos algoritmos para un mismo par |
| `find_nodes` | Busca nodos del grafo por nombres de calles o intersecciones |
| `search_documents` | Recupera fragmentos relevantes del corpus documental |

### Herramientas de rutas

`plan_route` y `get_route_metrics` utilizan las implementaciones de A* y Dijkstra del proyecto. Las distancias y métricas se obtienen de los algoritmos; no son estimaciones generadas por el modelo de lenguaje.

El modelo recibe una representación compacta del resultado para reducir el tamaño del contexto. La traza conserva información más completa de la ejecución, incluyendo el camino cuando corresponde.

`find_nodes` facilita consultas basadas en nombres de calles. Su búsqueda se limita a los nodos de la red local; no funciona como un servicio general de geocodificación para cualquier dirección de Medellín.

### Herramienta documental

`search_documents` consulta el índice local y devuelve fragmentos con sus metadatos y puntuaciones de similitud.

Cuando no encuentra contenido que supere el umbral configurado, comunica la ausencia de resultados relevantes.

## 5. Preparación del índice RAG

El corpus está compuesto por cuatro documentos oficiales del Metro de Medellín, preparados como 23 registros por página PDF o documento web.

### Fragmentación

Los registros se dividen en fragmentos con:

| Parámetro | Valor |
|---|---:|
| Tamaño máximo del fragmento | 140 palabras |
| Solapamiento | 40 palabras |
| Fragmentos del índice incluido | 97 |

El solapamiento conserva parte del contexto entre fragmentos consecutivos.

Cada fragmento mantiene información que permite identificar su procedencia:

- Documento de origen.
- Título.
- URL.
- Página, cuando corresponde.
- Fecha de captura.
- Nota de versión.

### Representación vectorial

El índice incluido, almacenado en `data/index/`, utiliza la siguiente configuración:

| Parámetro | Valor |
|---|---|
| Backend | `hashing` |
| Identificador de representación | `hashing-tfidf-4096` |
| Dimensiones | 4096 |
| Umbral mínimo de similitud | 0,07 |

Esta representación utiliza hashing de términos y ponderación TF-IDF. Es una recuperación **léxica**, basada en características del texto, y no un modelo de embeddings semánticos.

El código también admite un backend con `sentence-transformers`, pero no es el utilizado por el índice incluido ni por la evaluación registrada.

### Recuperación

La consulta se convierte en un vector compatible con el índice. Se calcula la similitud coseno con los fragmentos y se seleccionan hasta los `k` resultados mejor puntuados que superen el umbral mínimo.

El índice conserva el backend, el modelo, los parámetros de fragmentación y el hash del corpus. Si detecta que el corpus cambió respecto a la versión indexada, solicita reconstruir el índice para evitar utilizar datos desactualizados.

## 6. Citas y tratamiento de las fuentes

Cada fragmento recuperado recibe un identificador temporal, como `S1`, `S2` o `S3`, válido durante la consulta.

El modelo recibe instrucciones para citar esos identificadores al elaborar afirmaciones documentales. La función `ask()` devuelve únicamente las fuentes recuperadas que aparecen citadas en la respuesta.

Las referencias a identificadores inexistentes no se incorporan a la lista de fuentes. Este control verifica la correspondencia entre las marcas y los fragmentos recuperados, pero no garantiza por sí solo que cada afirmación esté correctamente respaldada.

Cuando los documentos no contienen la información solicitada, las instrucciones exigen comunicar esa limitación en lugar de completar la respuesta de memoria.

### Documentos históricos y contradicciones

El corpus incluye documentos de distintas fechas que pueden contener condiciones diferentes.

Las notas de versión permiten identificar ese contexto. Ante una contradicción, el agente recibe instrucciones para señalarla y mostrar las fuentes correspondientes, sin presentar como vigente una disposición que no pueda verificar.

Los fragmentos se tratan como información documental y no como instrucciones que puedan modificar el comportamiento del sistema.

## 7. Manejo de errores

El agente contempla situaciones como:

- Configuración incompleta del proveedor.
- Fallos de conexión, autenticación o límites de uso.
- Respuestas inesperadas del modelo.
- Argumentos inválidos para una herramienta.
- Nodos inexistentes o ausencia de ruta dirigida.
- Información documental insuficiente.
- Agotamiento del límite de iteraciones.

Los errores de las herramientas se devuelven al modelo como observaciones para que pueda explicarlos al usuario. Los errores del proveedor se comunican mediante una respuesta comprensible y el campo `error`.

La interfaz mantiene disponible el cálculo de rutas aunque el proveedor del chat no responda.

## 8. Evaluación del agente

Se definieron 12 preguntas fijas en `tests/agent_questions.json`:

| Categoría | Cantidad |
|---|---:|
| Consultas de rutas | 4 |
| Consultas documentales con cita | 3 |
| Consultas que encadenan ruta y métricas | 2 |
| Errores o información ausente | 3 |
| **Total** | **12** |

El script de evaluación se encuentra en:

```text
scripts/agent/run_agent_eval.py
```

Los resultados se almacenan en:

- `results/agent_eval.csv`
- `results/agent_eval_summary.json`

Se registra la herramienta esperada y la utilizada, las citas, la comprobación numérica y el resultado de cada caso.

Los criterios que no corresponden a una pregunta se marcan como `n/a` y no se contabilizan como aciertos. Las preguntas de error o información ausente requieren revisión manual según el criterio establecido.

### Resultados registrados

La corrida documentada utilizó Groq con el modelo `openai/gpt-oss-120b` y el índice léxico `hashing`.

| Criterio | Resultado |
|---|---:|
| Preguntas acertadas automáticamente | 6 de 9 |
| Uso de herramientas esperadas | 9 de 10 casos aplicables |
| Citas de documentos esperados | 3 de 3 |
| Comprobación numérica automática | 3 de 6 |
| Error de herramienta esperado | 1 de 1 |
| Revisión manual registrada en el resumen | 3 de 3 |

Estos resultados corresponden a una corrida concreta; no representan una garantía de comportamiento para todas las consultas.

### Interpretación de los fallos

En dos casos, el agente presentó las distancias correctas utilizando separadores de miles o formatos decimales que el verificador automático no reconoció. Por ello, algunos fallos de comprobación numérica corresponden al evaluador y no a un cálculo incorrecto de las herramientas.

En otro caso, el agente comunicó cifras correctas de nodos expandidos, pero no llamó a una de las herramientas exigidas por el caso de prueba. Este resultado se conserva como un incumplimiento del criterio de evaluación.

El resumen registra que las tres preguntas revisadas manualmente cumplieron sus criterios de manejo de errores o información ausente.

## 9. Ejecución y comprobación

### Aplicación

Con el entorno virtual activo y las dependencias instaladas:

```bash
python -m streamlit run app.py
```

El chat requiere completar la configuración del proveedor en `.env`.

Una consulta documental de ejemplo es:

> ¿Qué condiciones debo cumplir para ingresar con mi mascota al Metro de Medellín? Responde según los documentos disponibles y cita las fuentes.

La respuesta debe revisarse junto con la lista de fuentes y la traza de herramientas. La presencia de `search_documents` permite comprobar que hubo una consulta al índice documental.

### Reconstrucción del índice incluido

```bash
python -m scripts.agent.build_index --backend hashing
```

Este comando reconstruye el índice léxico a partir del corpus local.

### Pruebas automáticas

```bash
python -m unittest discover -s tests -t .
```

Las pruebas con clientes simulados no requieren una clave real.

### Evaluación con el proveedor

```bash
python -m scripts.agent.run_agent_eval --sleep 25
```

La evaluación real requiere credenciales, conexión y una configuración compatible con el proveedor. Las pausas reducen la frecuencia de solicitudes, pero no garantizan evitar todos los límites de uso.

En la integración revisada, `app.py` configura un `User-Agent` específico para las solicitudes. Los scripts independientes no pasan por esa configuración, por lo que deben comprobarse los encabezados del cliente si el proveedor devuelve HTTP 403.

## 10. Limitaciones

- La calidad de las respuestas depende del modelo y de las instrucciones; una temperatura de cero no garantiza respuestas idénticas.
- El corpus contiene pocos documentos y parte de su contenido es histórico.
- La recuperación léxica puede fallar con sinónimos o consultas que no compartan términos con los documentos.
- Una similitud alta no demuestra que un fragmento responda completamente la pregunta.
- Las citas permiten rastrear las fuentes, pero no certifican la vigencia ni la interpretación correcta de sus contenidos.
- Las rutas utilizan únicamente distancia y se limitan al extracto vial disponible.
- Puede existir una ruta en un sentido y no en el contrario.
- Los documentos del Metro no están conectados al grafo como una red de estaciones o recorridos ferroviarios.
- Las restricciones del proveedor pueden producir demoras o errores temporales.
- La evaluación utiliza un conjunto pequeño de preguntas y una sola corrida registrada.

## 11. Conclusión

El agente integra consultas documentales con herramientas que operan sobre datos reales del proyecto. La recuperación aporta fragmentos con procedencia identificable y las herramientas proporcionan rutas y métricas calculadas mediante los algoritmos implementados.

La traza de ejecución y las fuentes permiten inspeccionar el proceso seguido para responder. La evaluación muestra funcionamiento en los casos probados y también identifica limitaciones del modelo, de la recuperación y del verificador automático que deben considerarse al interpretar los resultados.
