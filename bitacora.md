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