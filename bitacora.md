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

- **Modelo:** Claude (`claude-opus-5-5`), vía la API de Anthropic con la
  librería `anthropic`. Esfuerzo `low`, por ser una tarea corta.
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