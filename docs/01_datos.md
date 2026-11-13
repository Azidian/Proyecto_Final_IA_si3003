# Informe técnico de datos viales y corpus documental

**Proyecto:** Copiloto de movilidad de Medellín  
**Responsable del componente:** Isabella Cadavid Posada  
**Componente:** Preparación de datos viales y corpus documental

## 1. Contexto y objetivo

El proyecto desarrolla un copiloto académico de movilidad para una zona del centro de Medellín. Integra la búsqueda de rutas de menor distancia mediante A* y Dijkstra con un asistente que consulta documentos oficiales del Metro de Medellín.

Este informe describe la obtención, transformación y validación de los datos geográficos, la construcción del grafo dirigido y la preparación del corpus documental. Estos recursos sirven de entrada a los componentes de enrutamiento, recuperación documental e interfaz.

## 2. Fuente y cobertura geográfica

La red vial se obtuvo de OpenStreetMap mediante su API de mapas. OpenStreetMap es una fuente abierta colaborativa; los datos seleccionados corresponden a vías reales ubicadas en Medellín, Colombia.

Se delimitó una zona pequeña del centro de la ciudad mediante las siguientes coordenadas WGS84:

| Límite | Coordenada |
|---|---:|
| Oeste | -75.574 |
| Sur | 6.247 |
| Este | -75.566 |
| Norte | 6.255 |

La consulta utiliza el orden oeste, sur, este y norte:

`-75.574,6.247,-75.566,6.255`

Se conservó el archivo original en formato OSM XML y un registro de procedencia con URL, fecha de captura, tamaño y hash SHA-256. Esta información permite rastrear el origen del archivo y comprobar su integridad.

Los datos geográficos se atribuyen a **OpenStreetMap contributors** bajo la licencia **Open Database License —ODbL 1.0—**.

## 3. Preparación de la red vial

El procesamiento transformó el extracto geográfico en un grafo dirigido y ponderado. Cada nodo representa un punto con coordenadas y cada arista representa un segmento vial con un sentido y una distancia.

Se aplicaron los siguientes criterios:

- Seleccionar categorías de vías admitidas para un modelo simplificado de automóvil.
- Excluir accesos explícitamente restringidos según la jerarquía de etiquetas utilizada.
- Excluir vías con sentidos reversibles, alternantes o valores de sentido no interpretados.
- Conservar únicamente segmentos cuyos dos extremos estén dentro del área delimitada.
- Eliminar autociclos y descartar nodos que no participen en los segmentos conservados.
- Interpretar las etiquetas de sentido de circulación, incluyendo `oneway=yes`, `oneway=-1` y las rotondas.
- Generar aristas en ambos sentidos para las vías bidireccionales.
- Mantener los identificadores de las aristas paralelas y su relación con la vía original.

Los cruces geométricos se conectan únicamente cuando las vías comparten un identificador de nodo de OpenStreetMap. La proximidad visual entre dos segmentos no se considera suficiente para establecer una conexión.

La red procesada contiene:

| Elemento | Cantidad |
|---|---:|
| Nodos | 696 |
| Aristas dirigidas | 858 |
| Pares de origen y destino para evaluación | 10 |

## 4. Cálculo de distancias y representación

El costo de cada arista corresponde a la distancia geográfica entre dos nodos consecutivos, calculada mediante la fórmula haversine.

Las distancias se almacenan en metros, sin redondearlas durante su preparación. El costo de una ruta corresponde a la suma de las distancias de sus aristas.

Se conservaron los nodos de geometría de las vías, no solamente las intersecciones. Por esta razón, una calle puede estar representada por varios segmentos cortos.

Los datos procesados se organizan en:

| Archivo | Contenido |
|---|---|
| `data/processed/nodes.json` | Identificadores y coordenadas de los nodos |
| `data/processed/edges.json` | Aristas dirigidas, distancias y atributos de las vías |
| `data/processed/metadata.json` | Procedencia, cobertura y metadatos del procesamiento |
| `data/test_pairs.json` | Pares de origen y destino para evaluación |

La función pública `load_graph()` carga estos archivos locales sin realizar solicitudes HTTP. El grafo ofrece acceso a los nodos, las aristas y las listas de adyacencia que utilizan los algoritmos de búsqueda.

La arquitectura separa las entidades de dominio, los casos de uso y los adaptadores de infraestructura. Esta organización permite modificar la obtención o el almacenamiento de los datos sin cambiar las reglas del grafo.

## 5. Controles de calidad y pares de prueba

Se comprobaron los siguientes aspectos:

- Identificadores de nodos y aristas válidos y sin duplicados.
- Existencia de los nodos de origen y destino de cada arista.
- Coordenadas dentro de rangos geográficos válidos.
- Distancias finitas y no negativas.
- Coherencia entre las distancias almacenadas y su cálculo geográfico.
- Correspondencia entre los archivos originales y sus hashes registrados.

También se seleccionaron diez pares distintos de origen y destino. Su alcanzabilidad se verificó mediante BFS, respetando la dirección de las aristas.

Esta búsqueda comprueba que exista un recorrido para cada par seleccionado; no calcula la ruta de menor distancia ni sustituye las implementaciones de A* y Dijkstra.

Los pares quedan almacenados para que ambos algoritmos se evalúen sobre los mismos casos y la comparación sea reproducible.

## 6. Análisis exploratorio

El análisis descriptivo identificó:

| Indicador | Resultado |
|---|---:|
| Componentes débilmente conexas | 2 |
| Nodos de la componente mayor | 694 |
| Nodos de la componente menor | 2 |
| Nodos sin aristas salientes | 14 |
| Longitud media de arista dirigida | Aproximadamente 18,96 m |

Una componente débilmente conexa agrupa nodos conectados al ignorar temporalmente la dirección de las aristas. Esto no garantiza que todos sus nodos puedan alcanzarse entre sí respetando los sentidos de circulación.

Se conservaron las componentes y los nodos sin salida como características de la red extraída. No se añadieron conexiones artificiales para unirlos.

Las estadísticas y verificaciones se encuentran en:

- `results/data_validation.json`
- `results/eda.json`

## 7. Preparación del corpus documental

El corpus se obtuvo del sitio oficial del Metro de Medellín y contiene:

1. Reglamento del Usuario.
2. Condiciones de uso de la funcionalidad Bici.
3. Documento de convivencia para el ingreso de bicicletas.
4. Captura de la página de información y servicios para usuarios.

Se conservaron los tres PDF y la página HTML originales, junto con sus textos extraídos y registros de procedencia.

La preparación produjo:

| Elemento | Cantidad |
|---|---:|
| Documentos | 4 |
| Registros por página PDF o documento web | 23 |
| Palabras extraídas | 9.287 |

Cada registro documental incluye el identificador del documento, título, URL, texto, página cuando corresponde, fecha de captura y nota de versión.

Los archivos se organizan así:

| Ruta | Contenido |
|---|---|
| `corpus/raw/` | Documentos originales |
| `corpus/text/` | Textos extraídos |
| `corpus/sources.csv` | Fuentes y registros de consulta |
| `corpus/manifest.json` | Metadatos, hashes y conteos |
| `corpus/documents.jsonl` | Registros de texto con información para citas |

Estos recursos sirven de entrada al componente de fragmentación, indexación y recuperación documental. La preparación del corpus es una etapa del sistema RAG, no la totalidad de su implementación.

## 8. Verificación y reproducibilidad

La preparación y validación local se ejecuta mediante:

```bash
python -m scripts.prepare_data
python -m scripts.prepare_corpus
python -m scripts.describe_data
python -m scripts.validate
