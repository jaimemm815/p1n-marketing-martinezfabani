# Bitácora del proyecto

## 7 de octubre de 2026

### Definición del proyecto

Se decidió desarrollar un sistema de publicidad contextual basado en PLN que
analice el contenido textual de una página web y permita recomendar publicidad
relacionada con su contexto.

### Selección del corpus

Se seleccionó el dataset `mteb/SpanishNewsClassification`, disponible en
Hugging Face.

El corpus contiene 2.048 documentos en español distribuidos de forma casi
equilibrada en 12 categorías.

La unidad de análisis elegida es el documento completo.

### Recuperación de categorías

El dataset seleccionado conserva las categorías mediante etiquetas numéricas.
Para recuperar sus nombres se cruzó una parte de los textos con el dataset
fuente `mteb/spanish_news`.

Se obtuvo el siguiente mapeo:

- 0: alimentation
- 1: astronomy
- 2: economy
- 3: fashion
- 4: medicine
- 5: military
- 6: motor
- 7: play
- 8: politics
- 9: religion
- 10: sport
- 11: tech

## 3 de noviembre de 2026 — Hito 2

### Ampliación del corpus con WikiHow

Para comprobar si el sistema generaliza más allá del texto periodístico (una de
las limitaciones señaladas en el Hito 1), se añadieron 500 artículos en español
de WikiHow. El corpus del Hito 2 (`data/hito2/corpus_hito2.csv`) queda en
2.548 documentos: 2.048 noticias y 500 artículos de WikiHow, identificados por la
columna `source_type`.

### Muestra de evaluación

Se extrajo una muestra de 100 textos (`data/hito2/muestra_evaluacion.csv`),
equilibrada por fuente: 50 de `news` y 50 de `wikihow`.

### Generación con IA generativa

- **Modelo:** Claude (`claude-opus-5-5`). El notebook puede llamarlo vía la API
  de Anthropic con la librería `anthropic` (esfuerzo `low`, por ser una tarea
  corta) o leer respuestas ya generadas (modo manual; ver entrada del 4 de
  noviembre).
- **API key:** se lee de un archivo `.env` local mediante `python-dotenv`. El
  archivo está en `.gitignore`, por lo que la clave no aparece en el notebook ni
  se sube a GitHub; `.env.example` sirve de plantilla.
- **Diseño en dos pasos:** el modelo asigna primero una **categoría contextual**
  y después genera un anuncio adecuado a esa categoría y al contenido del texto.
- **Categorías:** se usan las 12 categorías originales del corpus en lugar de
  las 9 categorías publicitarias planteadas inicialmente. Así la clasificación
  se evalúa con las mismas etiquetas del Hito 1. Contrapartida: algunos textos de
  WikiHow (p. ej. «cómo abrir las puertas de un tren») no encajan bien en ninguna,
  y se pide al modelo que elija la más cercana.
- **Prompt (`crear_prompt`):** incluye la lista de categorías con una breve
  descripción en español y exige un formato fijo (`CATEGORIA:` / `ANUNCIO:`) para
  extraer los campos automáticamente con expresiones regulares.
- **Proceso:** prueba con 1 texto, después con 10 y finalmente con los 100. Si una
  respuesta no contiene una categoría de la lista permitida, se vuelve a pedir
  (hasta 3 intentos). Al final se valida que todas las categorías generadas
  pertenecen a la lista.

### Evaluación humana

Dos anotadores evalúan por separado los 100 casos, cada uno en su propio archivo
(`anotador_1.csv`, `anotador_2.csv`), sin ver las respuestas del otro:

- `categoria_correcta`: 1 = sí, 0 = no.
- `encaje_anuncio`: 2 = encaja claramente, 1 = encaje parcial, 0 = no encaja.
- `comentario`: opcional.

Métricas previstas, en total y por fuente (`news` frente a `wikihow`):
porcentaje de acuerdo y Cohen's Kappa para ambas variables (para el encaje,
además, Kappa ponderada cuadrática por ser una escala ordinal), porcentaje de
categorías correctas según ambos anotadores y encaje medio.

## 4 de noviembre de 2026 — Generación de la muestra

### Generación manual con un LLM

No se disponía de API key, así que se optó por la alternativa admitida de
generar los anuncios manualmente con un LLM. Se usó Claude (Opus 5.5) en una
sesión interactiva de Claude Code y se aplicó a cada uno de los 100 textos
exactamente el mismo prompt de `crear_prompt`. Los textos se procesaron en 4
lotes de 25.

Las respuestas, en el mismo formato `CATEGORIA:` / `ANUNCIO:`, se guardaron en
`data/hito2/respuestas_manuales.csv`. Cuando no encuentra API key, el notebook
funciona en modo `manual`: lee ese archivo en lugar de llamar a la API. La
extracción, la validación y la evaluación son idénticas en ambos modos, y basta
con añadir una clave en `.env` para regenerar los resultados vía API.

### Resultados de la generación

- 100/100 respuestas con el formato esperado y con una categoría de la lista
  permitida; ningún reintento necesario.
- Anuncios de entre 20 y 29 palabras, sin anuncios repetidos.
- Resultados en `data/hito2/muestra_generada.csv`.

Distribución de las categorías generadas:

| Categoría | news | wikihow | Total |
|---|---|---|---|
| medicine | 4 | 12 | 16 |
| play | 5 | 9 | 14 |
| tech | 7 | 7 | 14 |
| fashion | 5 | 8 | 13 |
| alimentation | 4 | 5 | 9 |
| economy | 1 | 6 | 7 |
| religion | 7 | 0 | 7 |
| sport | 5 | 1 | 6 |
| politics | 5 | 0 | 5 |
| astronomy | 3 | 0 | 3 |
| military | 2 | 1 | 3 |
| motor | 2 | 1 | 3 |

Las noticias se reparten entre las 12 categorías. Los textos de WikiHow se
concentran en `medicine`, `play`, `fashion` y `tech`, y no aparecen nunca en
`politics`, `religion` ni `astronomy`. `play` funciona como categoría de
refugio para manualidades y bricolaje (p. ej. decoupage o piedra artificial), y
hay asignaciones forzadas, como «cómo abrir las puertas de un tren» → `motor`.
Es una primera señal de que la taxonomía periodística encaja peor con WikiHow,
algo que la evaluación humana debe confirmar.

### Archivos de anotación

Se crearon `data/hito2/anotador_1.csv` y `data/hito2/anotador_2.csv`, vacíos e
independientes. Cada anotador rellena el suyo sin ver el del otro. El notebook
no los sobrescribe si ya existen y no calcula las métricas
(`resultados_evaluacion.csv`) hasta que ambos estén completos.