# Declaración de uso de IA generativa de Isabella

**Proyecto:** Copiloto de movilidad de Medellín  
**Responsable:** Isabella Cadavid Posada  
**Participación:** Datos viales, corpus documental, revisión de documentación y comprobación de la aplicación integrada  
**Herramienta utilizada:** ChatGPT con Codex

## 1. Propósitos de uso

Utilicé ChatGPT con Codex como apoyo para preparar los datos viales y el corpus documental. La herramienta propuso una estructura para organizar el código, generó código Python y ayudó a preparar los scripts de descarga y transformación, las pruebas automatizadas y los borradores de la documentación.

También la consulté para recibir explicaciones sobre la representación de la red mediante nodos y aristas, la organización de las capas del proyecto, la configuración del entorno de Python en mi Mac y la interpretación de las validaciones.

Posteriormente, utilicé la herramienta como apoyo para ejecutar la aplicación integrada, configurar el proveedor del modelo de lenguaje, diagnosticar errores de conexión, comprender el funcionamiento del agente con RAG y preparar una demostración de su uso.

La asistencia incluyó generación de código y textos, explicaciones técnicas y orientación para resolver errores. No se limitó a correcciones de redacción.

## 2. Preparación y procedencia de los datos

Los datos de la red vial se descargaron de OpenStreetMap y los documentos del corpus se obtuvieron del sitio oficial del Metro de Medellín. Se conservaron los archivos originales, sin generar coordenadas ni documentos sintéticos para reemplazarlos.

OpenStreetMap es una fuente abierta colaborativa. Los documentos del Metro son fuentes institucionales oficiales y constituyen el contenido documental consultado por el asistente.

Los registros de procedencia y manifiestos incluyen las fuentes y los hashes de los archivos. La fuente permite rastrear de dónde se obtuvo cada archivo y el hash permite comprobar si una copia coincide con la versión registrada.

La IA apoyó la preparación de los procedimientos de descarga, transformación y validación. La procedencia de los datos se documentó mediante los archivos originales y sus metadatos, no mediante afirmaciones del modelo.

## 3. Apoyo en la arquitectura y documentación

Utilicé la herramienta para organizar y explicar la separación entre:

- Entidades de dominio, como nodos, aristas y grafo.
- Casos de uso de carga y comprobación de conectividad.
- Adaptadores para archivos, descargas y extracción documental.
- Comandos de preparación, validación y demostración.

También recibí apoyo para redactar y revisar el README, la descripción de los datos, las fuentes, la arquitectura, las limitaciones y esta declaración.

En la revisión de la documentación integrada, consulté explicaciones sobre el ciclo de llamadas a herramientas, la recuperación documental y los resultados de evaluación registrados. Esta revisión documental no implica que haya implementado personalmente todos los componentes descritos.

## 4. Pruebas y corrección de precisión numérica

Durante la preparación del componente de datos se ejecutaron 13 pruebas unitarias y de integración. Las verificaciones incluyeron:

- Carga del grafo desde archivos locales sin solicitudes HTTP.
- Validación de coordenadas, distancias e identificadores.
- Interpretación de sentidos de circulación y restricciones básicas de acceso.
- Alcanzabilidad dirigida de los 10 pares de origen y destino.
- Reconstrucción de los datos a partir del archivo original.
- Detección de duplicados y alteraciones de los originales.

Al ejecutar las pruebas en mi Mac apareció una diferencia en el último decimal de una distancia reconstruida. Utilicé la asistencia de IA para diagnosticar el problema y ajustar la prueba, que inicialmente exigía igualdad byte por byte.

La comparación corregida conserva la igualdad de nodos, topología, metadatos y pares, y permite una tolerancia absoluta de `1e-8` metros en las distancias.

Además, el comando de validación comprobó la integridad de los archivos del corpus y la correspondencia entre los textos extraídos y los registros documentales.

## 5. Comprobaciones personales del componente de datos

Al intentar ejecutar los comandos en mi Mac, la terminal mostró `zsh: command not found: python`. Comprobé que el equipo reconocía `python3`, que reportaba la versión `3.15.0rc2`, y ejecuté:

```bash
python3 --version
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
