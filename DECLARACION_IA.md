# Declaración de uso de IA generativa

**Proyecto:** Copiloto de movilidad de Medellín  
**Asignatura:** SI3003 – Inteligencia Artificial  
**Integrantes que presentan esta declaración:** Isabella Cadavid Posada y Wendy

Este documento describe las herramientas de IA generativa utilizadas por cada declarante, sus propósitos y las comprobaciones realizadas personalmente.

La asistencia de IA incluyó explicaciones, generación de código, preparación de pruebas, documentación y diagnóstico de errores. Su utilización no sustituye la responsabilidad de comprender el trabajo presentado ni la revisión de sus resultados.

## 1. Isabella Cadavid Posada

### 1.1 Herramienta utilizada

**ChatGPT con Codex**, como asistente de explicación, programación, documentación y diagnóstico.

### 1.2 Propósitos de uso

Utilicé la herramienta para:

- Recibir explicaciones sobre la representación de la red vial mediante nodos, aristas, coordenadas y distancias.
- Proponer una estructura de código que separara dominio, casos de uso y adaptadores de infraestructura.
- Apoyar la generación y organización del código de descarga, transformación y carga de los datos de OpenStreetMap.
- Preparar la extracción de texto de los documentos oficiales del Metro de Medellín, conservando su procedencia.
- Generar pruebas automatizadas y comprender sus resultados.
- Diagnosticar una diferencia de precisión numérica entre entornos y ajustar la comparación de las distancias reconstruidas.
- Elaborar y revisar borradores del README, los informes técnicos, las fuentes y las limitaciones.
- Configurar el entorno virtual de Python y recibir orientación sobre Git y GitHub.
- Ejecutar la aplicación integrada y comprender sus controles, métricas y funcionamiento documental.
- Configurar el proveedor LLM y diagnosticar un error de certificados HTTPS en macOS.
- Preparar el guion de la demostración del proyecto.

La asistencia incluyó generación de código y textos; no se limitó a correcciones de estilo.

### 1.3 Procedencia de los datos

Los datos geográficos se obtuvieron de OpenStreetMap y los documentos del corpus proceden del sitio oficial del Metro de Medellín.

Se conservaron los originales y los registros de procedencia, incluyendo fuentes y hashes. No se generaron coordenadas ni documentos sintéticos para reemplazar datos reales.

OpenStreetMap es una fuente abierta colaborativa. Los documentos del Metro son materiales institucionales oficiales y conservan su autoría.

### 1.4 Comprobaciones personales de datos y corpus

Creé y activé el entorno virtual en mi Mac e instalé las dependencias del proyecto.

Ejecuté personalmente:

```bash
python -m scripts.demo_data
python -m scripts.validate
python -m unittest discover -s tests -v
```

Comprobé:

- La carga de **696 nodos y 858 aristas dirigidas**.
- La validación de **10 pares de origen y destino**.
- La disponibilidad de **cuatro documentos**.
- El resultado **`status: ok`** del validador.
- La aprobación de las **13 pruebas del componente de datos**, después del ajuste de precisión numérica.

También ejecuté:

```bash
python -m scripts.prepare_data
python -m scripts.prepare_corpus
python -m scripts.describe_data
```

Confirmé que finalizaran correctamente y que el corpus produjera **23 registros documentales y 9.287 palabras extraídas**. Posteriormente repetí las pruebas y la validación con resultados correctos.

La comparación de reconstrucción utiliza una tolerancia absoluta de `1e-8` metros para las distancias. Mantiene las comprobaciones de identificadores, topología, metadatos y pares de prueba.

### 1.5 Comprobación de la aplicación y del RAG

Inicié la aplicación mediante:

```bash
python -m streamlit run app.py
```

Configuré localmente el proveedor Groq mediante las variables `LLM_API_KEY`, `LLM_BASE_URL` y `LLM_MODEL`, utilizando el modelo `openai/gpt-oss-120b`.

Durante la configuración recibí apoyo para diagnosticar el error `SSL: CERTIFICATE_VERIFY_FAILED` y reparar los certificados de Python sin desactivar la verificación HTTPS.

Realicé la consulta:

> ¿Qué condiciones debo cumplir para ingresar con mi mascota al Metro de Medellín? Responde según los documentos disponibles y cita las fuentes.

Observé la respuesta, sus referencias y la traza de herramientas. Comprobé que se utilizó `search_documents`, con una consulta relacionada con mascotas y un resultado de estado `ok`.

Esta comprobación demuestra el funcionamiento observado en esa consulta; no garantiza que todas las respuestas del agente sean correctas.

### 1.6 Límites de mi revisión

Mi revisión incluyó la preparación y carga de datos, la ejecución de comandos, la comprobación de resultados y una consulta documental desde la aplicación.

No realicé un cotejo página por página de todo el corpus. Los hashes permiten verificar integridad, pero no certifican la exactitud, fidelidad completa o vigencia del contenido.

Las 13 pruebas mencionadas corresponden al componente de datos y no deben presentarse como una evaluación completa del agente, los algoritmos y la interfaz.

## 2. Wendy

### 2.1 Herramienta utilizada

**Gemini**, como asistente de sintaxis, programación y redacción técnica.

### 2.2 Propósitos de uso

Utilicé la herramienta para:

- Recibir apoyo con la sintaxis de Python y el manejo de colas de prioridad mediante `heapq`.
- Preparar código de apoyo para A* y Dijkstra, respetando la restricción de no delegar el enrutamiento a NetworkX u OSMnx.
- Generar la estructura inicial de las pruebas de rutas y sus casos de error.
- Estructurar la exportación de métricas a CSV.
- Revisar la redacción de la explicación de la heurística y los resultados experimentales.

La asistencia se utilizó como apoyo al desarrollo; las decisiones y comprobaciones descritas a continuación forman parte de mi revisión del componente.

### 2.3 Decisiones técnicas revisadas

**Conteo de nodos expandidos.** Se definió como expandido un nodo cuando se extrae de la cola y se examinan sus aristas salientes. Las inserciones y las entradas obsoletas no se contabilizan como expansiones adicionales.

**Desempate en la cola.** Se incorporó un contador incremental en las entradas para mantener un criterio de desempate estable.

**Heurística.** A* utiliza `src.domain.geo.haversine_m` para estimar la distancia restante al destino. Como las aristas también tienen costos calculados mediante haversine, la desigualdad triangular permite justificar que esta estimación no sobreestima el costo de un recorrido por el grafo.

**Contrato de respuesta.** Se verificó que `plan_route` devolviera los campos:

- `status`
- `path`
- `distance_m`
- `expanded_nodes`
- `elapsed_ms`
- `error`

También se contemplaron nodos inexistentes, ausencia de ruta, origen igual a destino y algoritmos no reconocidos.

### 2.4 Comprobaciones personales declaradas

Según mi registro de trabajo:

- Ejecuté las siete pruebas del componente de enrutamiento, incluyendo la comparación de costos entre A* y Dijkstra y los casos de error.
- Ejecuté el script de evaluación e inspeccioné `results/routes.csv`.
- Comprobé que los diez pares evaluados registraran `same_optimal_distance == True`.
- Revisé la implementación de la cola de prioridad, la reconstrucción de caminos y la medición con `perf_counter`.
- Comprobé que el cálculo de rutas no dependiera de bibliotecas que resolvieran los algoritmos por el equipo.

Los resultados registrados muestran una reducción de nodos expandidos con A* de hasta **62,08 %**. Este resultado no implica que A* sea más rápido en todas las ejecuciones.

### 2.5 Límites de la evaluación

La evaluación utiliza diez pares sobre un extracto pequeño de la red vial. Sus resultados no demuestran el mismo comportamiento para todos los recorridos de Medellín.

Los tiempos corresponden a ejecuciones concretas y pueden variar según el equipo y las condiciones de ejecución. La reducción de nodos expandidos y el tiempo de cálculo son métricas distintas.

## 3. Procedencia de los componentes integrados

El proyecto presentado integra componentes desarrollados durante el trabajo compartido del equipo.

El agente, el sistema de recuperación y la interfaz se incorporaron desde esos componentes. La revisión de su documentación, la ejecución local o su presentación en el video no equivalen a atribuirse su implementación inicial.

Las declaraciones originales de quienes desarrollaron esos componentes deben conservarse como antecedentes del proyecto. Las herramientas de IA declaradas por otras participantes no se presentan aquí como herramientas utilizadas personalmente por Isabella o Wendy.

## 4. IA utilizada por el producto

El uso de IA durante el desarrollo se distingue del modelo que funciona dentro de la aplicación.

| Uso | Descripción |
|---|---|
| Asistencia al desarrollo | Explicaciones, código, pruebas, documentación y diagnóstico mediante las herramientas declaradas por cada participante |
| Modelo en ejecución | Proveedor LLM configurado en `.env`, utilizado por el agente para generar respuestas y solicitar herramientas |
| Recuperación documental | Consulta del índice local para proporcionar fragmentos del corpus como contexto |
| Cálculo de rutas | Ejecución de A* y Dijkstra sobre el grafo, sin delegar los cálculos al modelo de lenguaje |

El índice incluido utiliza recuperación léxica mediante hashing y TF-IDF. No se declara como un índice de embeddings semánticos.

## 5. Credenciales y seguridad

Las claves reales deben almacenarse únicamente en `.env`, fuera del control de versiones. `.env.example` debe contener campos vacíos o valores de configuración que no sean secretos.

Durante la preparación de la entrega, GitHub detectó una clave real en `.env.example` y bloqueó la subida. Se identificó la necesidad de retirar el secreto del archivo y del commit pendiente, y de revocar la credencial afectada.

Este documento no da por completadas esas acciones únicamente por haberse recibido las instrucciones; su ejecución debe comprobarse antes de cerrar la entrega.

## 6. Limitaciones y responsabilidad

Los datos y documentos proceden de fuentes identificadas, pero pueden contener errores, omisiones o información histórica.

Las pruebas automatizadas no garantizan por sí solas la comprensión del código, la vigencia documental ni la corrección de todas las respuestas del agente.

Las citas permiten rastrear fragmentos recuperados; no certifican automáticamente que una interpretación sea correcta. Las respuestas deben contrastarse con las fuentes cuando sea necesario.

Cada declarante es responsable de que su sección refleje las herramientas que realmente utilizó, las acciones que ejecutó y el alcance de sus comprobaciones. La asistencia de IA no sustituye esa responsabilidad.
