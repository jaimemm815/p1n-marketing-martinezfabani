# Publicidad contextual basada en PLN

Práctica 1 — Marketing · CUNEF Universidad

Sistema de publicidad contextual que analiza el contenido textual de una página
web y permite recomendar publicidad relacionada con su temática. Este primer
hito cubre la selección del corpus, el análisis exploratorio, el pipeline de
preprocesamiento y una primera caracterización léxica de las categorías.

## Corpus

- **Dataset:** [`mteb/SpanishNewsClassification`](https://huggingface.co/datasets/mteb/SpanishNewsClassification) (Hugging Face)
- **Volumen:** 2.048 documentos en español
- **Unidad de análisis:** documento completo
- **Categorías:** 12, con una distribución prácticamente equilibrada (~170 documentos por categoría)

El corpus conserva las categorías como etiquetas numéricas sin su nombre. Para
recuperarlas se cruzaron los textos con el dataset fuente
[`mteb/spanish_news`](https://huggingface.co/datasets/mteb/spanish_news),
obteniendo el siguiente mapeo:

| Label | Categoría | Label | Categoría |
|---|---|---|---|
| 0 | alimentation | 6 | motor |
| 1 | astronomy | 7 | play |
| 2 | economy | 8 | politics |
| 3 | fashion | 9 | religion |
| 4 | medicine | 10 | sport |
| 5 | military | 11 | tech |

## Estructura del repositorio

```
.
├── data/
│   ├── raw/          # Corpus original (train-00000-of-00001.parquet)
│   ├── processed/    # Datos derivados (SpanishNewsClassification.xlsx)
│   └── sample/       # Muestra de 20 documentos para inspección rápida
├── notebooks/
│   └── 01_hito1.ipynb   # EDA, preprocesamiento y análisis TF-IDF
├── bitacora.md       # Registro cronológico de decisiones del proyecto
├── requirements.txt  # Dependencias con versiones fijadas
└── README.md
```

## Instalación

Requiere Python 3.11 o superior.

```bash
git clone https://github.com/jaimemm815/p1n-marketing-martinezfabani.git
cd p1n-marketing-martinezfabani

python -m venv .venv
.venv\Scripts\activate        # Windows
# source .venv/bin/activate   # macOS / Linux

pip install -r requirements.txt
```

El modelo de spaCy para español (`es_core_news_sm`) se instala junto con el
resto de dependencias desde `requirements.txt`. Si fuese necesario instalarlo
por separado:

```bash
python -m spacy download es_core_news_sm
```

## Ejecución

```bash
jupyter lab
```

Abrir `notebooks/01_hito1.ipynb` y ejecutar las celdas en orden. El notebook lee
el corpus desde `data/raw/` mediante rutas relativas, por lo que no requiere
configuración adicional.

## Metodología

### 1. Análisis exploratorio

Revisión de dimensiones, tipos, valores nulos, textos vacíos y duplicados;
distribución de categorías; y análisis de la longitud de los documentos por
categoría.

### 2. Preprocesamiento

El pipeline (`preprocess_text`) se aplica en este orden:

1. **Tokenización** con spaCy (`es_core_news_sm`), que separa correctamente las
   unidades léxicas del español.
2. **Conversión a minúsculas**, para evitar que una misma palabra se trate como
   términos distintos.
3. **Eliminación de stopwords**, conservando las negaciones `no`, `nunca`,
   `jamás` y `sin` por su relevancia para análisis posteriores de tono.
4. **Lematización**, preferida al stemming por mantener formas
   lingüísticamente interpretables.
5. **Eliminación de puntuación y espacios**. Las cifras se conservan, ya que
   aportan información en categorías como economía, deporte o medicina.

### 3. Vectorización

Representación TF-IDF (`TfidfVectorizer`, `min_df=3`) sobre el texto procesado,
y extracción de los términos más característicos de cada categoría a partir del
TF-IDF medio.

## Resultados principales

**Impacto del preprocesamiento**

| Métrica | Antes | Después | Reducción |
|---|---|---|---|
| Tokens totales | 1.396.903 | 689.489 | −50,6 % |
| Tamaño del vocabulario | 136.252 | 66.091 | −51,5 % |

**Longitud de los documentos**

La distribución presenta una cola derecha pronunciada: percentil 90 en 1.215
palabras, percentil 95 en 1.559 y percentil 99 en 2.769. Solo 3 documentos
superan las 5.000 palabras; el más extremo pertenece a `medicine` con 29.746.

**Separabilidad léxica**

La comparación de los 15 términos con mayor TF-IDF de `sport` y `tech` muestra
un solapamiento de solo el 13,3 % (2 de 15 términos, `no` y `él`). En `sport`
dominan términos de competición (`partido`, `equipo`, `jugador`, `gol`), y en
`tech` términos de producto y servicio (`dispositivo`, `apple`, `usuario`,
`google`). Esta diferenciación es una primera señal favorable para la
clasificación temática automática.

## Limitaciones

- **Sesgo de fuente:** el corpus es periodístico, por lo que los resultados
  pueden no generalizar a blogs, tiendas online, foros o páginas corporativas,
  que son destinos habituales de la publicidad contextual.
- **Distribución artificial:** las categorías están prácticamente balanceadas,
  lo que resulta útil para el modelado pero no refleja la distribución real de
  contenidos en la web.
- **Calidad del texto:** se detectan términos residuales en inglés (`the`,
  `of`) y tokens anómalos (`|`), indicio de documentos no normalizados.
- **Granularidad:** las 12 categorías son amplias; la publicidad contextual
  real suele requerir una taxonomía más fina.

## Autores

- Jaime Martínez Martínez
- Isabella Fabani

CUNEF Universidad · Curso 2026-2027