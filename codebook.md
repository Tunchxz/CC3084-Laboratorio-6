# Codebook

## Convenciones Utilizadas

**Nivel de medición**

Se usa el vocabulario estándar más dos categorías necesarias:

| Nivel             | Qué significa                                              | Operaciones válidas                     |
| ----------------- | ---------------------------------------------------------- | --------------------------------------- |
| **Identificador** | Distingue una entidad. No es una categoría ni admite orden | Comparar igualdad, unir tablas          |
| **Nominal**       | Categorías sin orden                                       | Contar, agrupar, calcular moda          |
| **Ordinal**       | Categorías con orden, sin distancia definida               | Lo anterior, más mediana y cuantiles    |
| **De razón**      | Numérica con cero absoluto                                 | Todas, incluidas razones y proporciones |
| **Texto libre**   | Contenido no estructurado                                  | Análisis de texto                       |

**Archivos**

El proyecto tiene tres archivos de entrada y once generados:

| Archivo                                          | Filas × columnas | Sección                                   |
| ------------------------------------------------ | ---------------- | ----------------------------------------- |
| `data/raw/youtube_videos.csv`                    | 293 × 20         | [1.1](#11-youtube_videoscsv)              |
| `data/raw/youtube_comments.csv`                  | 406 × 17         | [1.2](#12-youtube_commentscsv)            |
| `data/raw/senticon.es.xml`                       | —                | [1.3](#13-senticonesxml)                  |
| `data/processed/videos_limpio.parquet`           | 293 × 25         | [2.1](#21-videos_limpioparquet)           |
| `data/processed/comentarios_limpio.parquet`      | 406 × 23         | [2.2](#22-comentarios_limpioparquet)      |
| `data/processed/comentarios_videos.parquet`      | 406 × 44         | [2.3](#23-comentarios_videosparquet)      |
| `data/processed/nodos_bipartita.csv`             | 351 × 15         | [3.1](#31-nodos_bipartitacsv)             |
| `data/processed/aristas_bipartita.csv`           | 343 × 5          | [3.2](#32-aristas_bipartitacsv)           |
| `data/processed/aristas_autor_autor.csv`         | 10,732 × 3       | [3.3](#33-aristas_autor_autorcsv)         |
| `data/processed/aristas_video_video.csv`         | 11 × 6           | [3.4](#34-aristas_video_videocsv)         |
| `data/processed/metricas_topologia.csv`          | 3 × 14           | [3.5](#35-metricas_topologiacsv)          |
| `data/processed/comunidades_nodos.csv`           | 351 × 16         | [3.6](#36-comunidades_nodoscsv)           |
| `data/processed/centralidades.csv`               | 351 × 9          | [3.7](#37-centralidadescsv)               |
| `data/processed/comentarios_sentimiento.parquet` | 406 × 13         | [4.1](#41-comentarios_sentimientoparquet) |

En los archivos generados **no se repiten las columnas heredadas**, se indica de dónde vienen y se documentan solo las nuevas.


## 1. Datos crudos (`data/raw/`)

### 1.1 `youtube_videos.csv`

**Unidad de observación:** un video de YouTube. **Llave primaria:** `video_id` (293 valores únicos, sin faltantes).

| Variable y descripción                                                                                                                            | Tipo y nivel                  | Unidades                     | Valores / rango / ejemplo                                                         |
| ------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------- | ---------------------------- | --------------------------------------------------------------------------------- |
| **`video_id`** — Identificador único del video en YouTube. Llave primaria y llave de unión con los comentarios                                    | Texto · Identificador         | —                            | 11 caracteres. Ej.: `-5puKGEqcUc`                                                 |
| **`title`** — Título del video tal como aparece publicado                                                                                         | Texto · Texto libre           | —                            | 274 títulos distintos. Ej.: `INSIVUMEH pronostica incremento de lluvias…`         |
| **`channel_name`** — Nombre visible del canal que publicó el video. Puede cambiar con el tiempo, por lo que **no debe usarse como identificador** | Texto · Nominal               | —                            | 97 valores. Ej.: `T13 Noticias Guatemala`                                         |
| **`channel_id`** — Identificador único del canal. Preferible a `channel_name` para construir nodos                                                | Texto · Identificador         | —                            | 97 valores. Ej.: `UCq0Cm-3SKthEySQc2JZBi1A`                                       |
| **`source_query`** — Consulta de búsqueda o canal con que se encontró el video. Describe **el muestreo, no el tema** del video                    | Texto · Nominal               | —                            | 21 valores. Ej.: `guatemala lluvias`, `@quorumgt/videos`                          |
| **`source_group`** — Estrategia de búsqueda que originó el registro                                                                               | Texto · Nominal               | —                            | `topic` (177), `official_gov` (105), `channel` (11)                               |
| **`dataset_sources`** — Archivos originales en los que apareció el video antes de integrar y eliminar duplicados                                  | Texto · Texto libre           | —                            | Nombres separados por ` \| `. 23 combinaciones                                    |
| **`channel_handle`** — Handle o nombre de usuario del canal                                                                                       | Texto · Identificador         | —                            | Formato `/@nombre`. Ej.: `/@T13NoticiasGuatemala`                                 |
| **`published_time`** — Tiempo transcurrido desde la publicación, **relativo al momento del scraping**. No es una fecha                            | Texto · Ordinal               | Expresión de tiempo relativo | 80 valores, 13 faltantes (4.4%). Ej.: `hace 2 días`                               |
| **`view_count_text`** — Visualizaciones en el formato mostrado por YouTube. Redundante frente a `view_count`                                      | Texto · De razón              | Vistas (como texto)          | 259 valores, 13 faltantes. Ej.: `2,390 vistas`                                    |
| **`description_snippet`** — Fragmento abreviado de la descripción mostrado en resultados de búsqueda                                              | Texto · Texto libre           | —                            | 236 valores, 25 faltantes (8.5%)                                                  |
| **`video_url`** — Dirección web completa del video                                                                                                | Texto · Identificador         | —                            | Ej.: `https://www.youtube.com/watch?v=-5puKGEqcUc`                                |
| **`query_hits`** — Lista de consultas con las que se recuperó el video. **Requiere convertirse de texto a lista** antes de usarse                 | Texto con estructura de lista | —                            | 26 valores. 5 videos tienen más de una. Ej.: `["guatemala lluvias"]`              |
| **`keywords`** — Palabras clave o etiquetas del video. **Requiere convertirse a lista**                                                           | Texto con estructura de lista | —                            | 106 valores; 162 videos tienen `[]`. Ej.: `["Guatemala", "Chapin tv"]`            |
| **`description`** — Descripción completa del video. Contiene hashtags, URL y llamados a la acción                                                 | Texto · Texto libre           | —                            | 234 valores, 26 faltantes (8.9%)                                                  |
| **`view_count`** — Visualizaciones observadas. **Variable recomendada** para análisis cuantitativos de popularidad                                | Entero · De razón             | Visualizaciones              | 2 a 8,190,449. Mediana: 1,175                                                     |
| **`publish_date`** — Fecha y hora de publicación en formato ISO 8601 con zona horaria                                                             | Texto · De intervalo          | Fecha y hora                 | Ej.: `2026-08-28T22:00:20-07:00`                                                  |
| **`upload_date`** — Fecha de carga. **Idéntica a `publish_date` en el 100% de los registros**                                                     | Texto · De intervalo          | Fecha y hora                 | Redundante. Se elimina en la limpieza                                             |
| **`category`** — Categoría de YouTube asignada al video                                                                                           | Texto · Nominal               | —                            | 11 valores. `News & Politics` (138), `People & Blogs` (66), `Entertainment` (48)… |
| **`owner_handle`** — Handle del propietario. **Idéntico a `channel_handle` en el 100%**                                                           | Texto · Identificador         | —                            | Redundante. Se elimina en la limpieza                                             |


### 1.2 `youtube_comments.csv`

**Unidad de observación:** un comentario principal. **Llave primaria:** `comment_id` (406 valores únicos, sin faltantes). **Llave foránea:** `video_id`.

| Variable y descripción                                                                                                                                                               | Tipo y nivel          | Unidades                     | Valores / rango / ejemplo                                                        |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------- | ---------------------------- | -------------------------------------------------------------------------------- |
| **`comment_id`** — Identificador único del comentario. Llave primaria                                                                                                                | Texto · Identificador | —                            | 406 valores. Ej.: `Ugw-J65a1iYL9hqhELh4AaABAg`                                   |
| **`video_id`** — Video en el que se publicó el comentario. Llave foránea hacia `youtube_videos.csv`                                                                                  | Texto · Identificador | —                            | Solo **19 valores distintos** de los 293 videos                                  |
| **`video_title`** — Título del video comentado. Redundante: se obtiene al unir por `video_id`                                                                                        | Texto · Texto libre   | —                            | 19 valores                                                                       |
| **`channel_name`** — Nombre del canal **dueño del video**. No es el autor del comentario                                                                                             | Texto · Nominal       | —                            | 8 valores. `Quorum`, `Noti7`, `Noticias Telemundo`…                              |
| **`channel_id`** — Identificador del canal dueño del video. **Distinto de `author_channel_id`**                                                                                      | Texto · Identificador | —                            | 8 valores                                                                        |
| **`author_name`** — Nombre visible del autor del comentario. Puede cambiar; **no usar como identificador**                                                                           | Texto · Nominal       | —                            | 332 valores. Ej.: `@MarcosCarillo-b1r`                                           |
| **`author_channel_id`** — Identificador único de la cuenta que comentó. **Es el que debe usarse como nodo de la red**                                                                | Texto · Identificador | —                            | 332 valores. Ej.: `UCdFlugHJJa4l3YqWuNRmvXw`                                     |
| **`text`** — Contenido completo del comentario. Fuente principal para tópicos, sentimiento y palabras frecuentes                                                                     | Texto · Texto libre   | —                            | 404 textos distintos (2 duplicados). De 1 a 1,525 caracteres                     |
| **`source_query`** — Consulta con que se obtuvieron los comentarios. Describe **el muestreo**                                                                                        | Texto · Nominal       | —                            | 6 valores. `@quorumgt/videos` aporta 231 de 406 (56.9%)                          |
| **`source_group`** — Tipo de fuente utilizada                                                                                                                                        | Texto · Nominal       | —                            | `channel` (231), `topic` (175)                                                   |
| **`dataset_sources`** — Archivos originales de comentarios en que apareció el registro                                                                                               | Texto · Texto libre   | —                            | 8 combinaciones, separadas por ` \| `                                            |
| **`author_handle`** — Handle del autor. Llega con **codificación porcentual** (`%C3%A9` en lugar de `é`) en el 3.4% de los casos                                                     | Texto · Identificador | —                            | 332 valores. Ej.: `/@MarcosCarillo-b1r`                                          |
| **`published_text`** — Tiempo desde la publicación del comentario, **relativo al scraping**. No es una fecha                                                                         | Texto · Ordinal       | Expresión de tiempo relativo | 39 valores. Ej.: `hace 6 meses`, `hace 2 años (editado)`                         |
| **`like_count_text`** — "Me gusta" recibidos, almacenados como texto. **Requiere limpieza y conversión a entero**                                                                    | Texto · De razón      | "Me gusta" (como texto)      | 31 valores. 189 registros (46.6%) contienen un espacio, que significa cero       |
| **`reply_count`** — Número de respuestas que recibió el comentario. **No identifica a los autores de esas respuestas, por lo que NO puede usarse para crear aristas entre usuarios** | Entero · De razón     | Respuestas                   | 0 a 7. El 92.6% tiene 0. Total: 51 respuestas                                    |
| **`is_pinned`** — Indica si el comentario fue fijado por el canal                                                                                                                    | Lógico · Nominal      | —                            | **Constante: `False` en los 406 registros.** Sin poder discriminante. Se elimina |
| **`viewer_rating`** — Valoración del usuario                                                                                                                                         | Numérica · —          | —                            | **Vacía en los 406 registros.** Inutilizable. Se elimina                         |


### 1.3 `senticon.es.xml`

Léxico de polaridad **ML-SentiCon**, usado en el Ejercicio 9 únicamente como punto de comparación frente al modelo finalmente elegido.

| Elemento y descripción                                            | Tipo y nivel          | Unidades            | Valores / rango / ejemplo                                       |
| ----------------------------------------------------------------- | --------------------- | ------------------- | --------------------------------------------------------------- |
| **`lemma`** — Lema en español con carga emocional                 | Texto · Identificador | —                   | 11,342 lemas. Ej.: `acertado`, `corrupto`                       |
| **`pol`** — Polaridad del lema                                    | Decimal · De razón    | Escala de polaridad | −1 (muy negativo) a +1 (muy positivo). Ej.: `admirable` = 0.906 |
| **`pos`** — Categoría gramatical del lema                         | Texto · Nominal       | —                   | `a` adjetivo, `n` sustantivo, `v` verbo, `r` adverbio           |
| **`std`** — Desviación estándar de la anotación entre evaluadores | Decimal · De razón    | Escala de polaridad | ≥ 0. Un valor alto indica desacuerdo entre anotadores           |
| **`layer`** — Capa de expansión del léxico                        | Entero · Ordinal      | Nivel               | 1 a 8. Las capas bajas son más precisas; las altas, más amplias |


## 2. Datos limpios (`data/processed/`)

### 2.1 `videos_limpio.parquet`

Producido por el Notebook 1. **Hereda las 20 variables** de `youtube_videos.csv`, con dos cambios: se eliminan `upload_date` y `owner_handle` por redundantes, y `keywords` y `query_hits` pasan de texto a lista real de Python. Los identificadores y handles quedan normalizados (Unicode NFKC, handles en minúsculas con `@` inicial).

Variables nuevas:

| Variable y descripción                                                                                                       | Tipo y nivel        | Unidades | Valores / rango / ejemplo                                                            |
| ---------------------------------------------------------------------------------------------------------------------------- | ------------------- | -------- | ------------------------------------------------------------------------------------ |
| **`texto_original`** — Concatenación de `title` y `description`, sin modificar. Base para la extracción de elementos         | Texto · Texto libre | —        | Ej.: `INSIVUMEH pronostica incremento de lluvias…`                                   |
| **`urls`** — Direcciones web extraídas del texto                                                                             | Lista de texto · —  | —        | 64 videos contienen alguna. Ej.: `[]`                                                |
| **`menciones`** — Menciones `@` extraídas, en minúsculas                                                                     | Lista de texto · —  | —        | Ej.: `[]`                                                                            |
| **`hashtags`** — Hashtags extraídos, en minúsculas y sin el símbolo `#`                                                      | Lista de texto · —  | —        | 586 ocurrencias en 122 videos. Ej.: `['guatemala', 'larondagt']`                     |
| **`emojis`** — Emojis distintos hallados, tanto Unicode como los propios de YouTube escritos como texto                      | Lista de texto · —  | —        | Ej.: `[]`                                                                            |
| **`tokens`** — Lemas del texto tras la limpieza completa: sin URL, menciones, emojis, puntuación, números ni palabras vacías | Lista de texto · —  | —        | 34.1 tokens por video en promedio. Ej.: `['insivumeh', 'pronosticar', 'incremento']` |
| **`texto_limpio`** — Los `tokens` unidos por espacios. Versión para conteo de palabras y bigramas                            | Texto · Texto libre | —        | Ej.: `insivumeh pronosticar incremento lluvia semana`                                |


### 2.2 `comentarios_limpio.parquet`

Producido por el Notebook 1. **Hereda las variables** de `youtube_comments.csv` menos tres: se eliminan `viewer_rating` (vacía), `is_pinned` (constante) y `text` (sustituida por `texto_original`). Los handles quedan normalizados y con la codificación porcentual decodificada.

Variables nuevas:

| Variable y descripción                                                                                                                                                                                                    | Tipo y nivel        | Unidades   | Valores / rango / ejemplo                                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------- | ---------- | ---------------------------------------------------------------------------- |
| **`like_count`** — `like_count_text` convertido a entero. Los valores vacíos se interpretan como **0**, porque YouTube omite el contador cuando no hay ninguno                                                            | Entero · De razón   | "Me gusta" | 0 a 405. Mediana: 1. El 46.6% tiene 0                                        |
| **`texto_original`** — Copia exacta del comentario publicado. **Se conserva para auditoría y para el análisis de sentimiento**, que necesita la puntuación, las mayúsculas y los emojis                                   | Texto · Texto libre | —          | Mediana de 96 caracteres; máximo 1,525                                       |
| **`urls`** — URL extraídas del comentario                                                                                                                                                                                 | Lista de texto · —  | —          | Solo 1 comentario contiene una                                               |
| **`menciones`** — Menciones `@` extraídas, en minúsculas                                                                                                                                                                  | Lista de texto · —  | —          | Solo 5 comentarios contienen alguna                                          |
| **`hashtags`** — Hashtags extraídos, sin el símbolo `#`                                                                                                                                                                   | Lista de texto · —  | —          | Solo 1 comentario contiene uno: `ineptobran`                                 |
| **`emojis`** — Emojis distintos, Unicode y de YouTube (`:face-blue-smiling:`)                                                                                                                                             | Lista de texto · —  | —          | 100 ocurrencias en 62 comentarios                                            |
| **`tokens`** — Lemas tras la limpieza completa y la eliminación de 533 palabras vacías                                                                                                                                    | Lista de texto · —  | —          | 9.5 tokens por comentario en promedio. Ej.: `['corrupto', 'amigo', 'viejo']` |
| **`texto_limpio`** — Los `tokens` unidos por espacios                                                                                                                                                                     | Texto · Texto libre | —          | Vocabulario total: 1,873 palabras distintas                                  |
| **`texto_limpio_vacio`** — Marca los comentarios que quedaron sin ningún token tras limpiar (solo emojis, una palabra vacía o una mención). **Se marcan en vez de eliminarse**, para no perder a esos 9 autores en la red | Lógico · Nominal    | —          | `True` en 10 comentarios (2.5%)                                              |


### 2.3 `comentarios_videos.parquet`

Unión de `comentarios_limpio` con `videos_limpio` por `video_id` (406 filas). **No tiene variables propias** (reúne las de ambos archivos).

Reglas de nombres:

- Las columnas que existían solo en uno de los dos conservan su nombre (`view_count`, `category`,
  `like_count`…).
- Las que existían en ambos llevan el sufijo **`_comentario`** o **`_video`** según su origen: `source_query_*`, `source_group_*`, `dataset_sources_*`, `texto_original_*`, `texto_limpio_*`, `tokens_*`, `urls_*`, `menciones_*`, `hashtags_*`, `emojis_*`.
- Se eliminaron del lado de los comentarios `video_title`, `channel_name` y `channel_id` por ser redundantes tras la unión; esos datos vienen del catálogo de videos.


## 3. Tablas de red (`data/processed/`)

Producidas por el Notebook 3. La red bipartita tiene 351 nodos (332 autores y 19 videos) y 343 aristas.

### 3.1 `nodos_bipartita.csv`

Una fila por nodo de la red bipartita. Las columnas exclusivas de un tipo de nodo quedan vacías en el otro.

| Variable y descripción                                                                                                             | Tipo y nivel          | Unidades        | Valores / rango / ejemplo                                   |
| ---------------------------------------------------------------------------------------------------------------------------------- | --------------------- | --------------- | ----------------------------------------------------------- |
| **`node_id`** — Identificador del nodo: es `author_channel_id` si es autor, o `video_id` si es video                               | Texto · Identificador | —               | 351 valores únicos                                          |
| **`tipo`** — Conjunto al que pertenece el nodo                                                                                     | Texto · Nominal       | —               | `autor` (332), `video` (19)                                 |
| **`etiqueta`** — Nombre visible para rotular gráficos: el handle del autor o el título del video. **Nunca sustituye al `node_id`** | Texto · Texto libre   | —               | Ej.: `@jorgemunoz-gd9et`, `Qué rico come tu diputado`       |
| **`grado`** — Número de vecinos. Para un autor, en cuántos videos comentó; para un video, cuántos autores distintos lo comentaron  | Entero · De razón     | Vecinos         | 1 a 128. El 97.3% de los autores tiene grado 1              |
| **`grado_ponderado`** — Suma de los pesos de sus aristas, es decir, número total de comentarios                                    | Entero · De razón     | Comentarios     | 1 a 161                                                     |
| **`n_comentarios`** — Comentarios asociados al nodo                                                                                | Entero · De razón     | Comentarios     | 1 a 161                                                     |
| **`n_videos`** — *(solo autores)* Videos distintos en que comentó                                                                  | Entero · De razón     | Videos          | 1 a 3. Vacío en los 19 videos                               |
| **`n_autores`** — *(solo videos)* Autores distintos que lo comentaron                                                              | Entero · De razón     | Autores         | 1 a 128. Vacío en los 332 autores                           |
| **`n_canales`** — *(solo autores)* Canales distintos en que comentó                                                                | Entero · De razón     | Canales         | 1 a 2                                                       |
| **`likes_totales`** — "Me gusta" acumulados por sus comentarios                                                                    | Entero · De razón     | "Me gusta"      | 0 a 1,697                                                   |
| **`respuestas_recibidas`** — *(solo autores)* Suma de `reply_count` de sus comentarios                                             | Entero · De razón     | Respuestas      | 0 a 7                                                       |
| **`canal`** — *(solo videos)* Nombre del canal que lo publicó                                                                      | Texto · Nominal       | —               | 8 valores                                                   |
| **`channel_id`** — *(solo videos)* Identificador del canal                                                                         | Texto · Identificador | —               | 8 valores                                                   |
| **`categoria`** — *(solo videos)* Categoría de YouTube                                                                             | Texto · Nominal       | —               | `News & Politics`, `Entertainment`, `Nonprofits & Activism` |
| **`view_count`** — *(solo videos)* Visualizaciones                                                                                 | Entero · De razón     | Visualizaciones | 52 a 304,089                                                |


### 3.2 `aristas_bipartita.csv`

Una fila por par (autor, video) con al menos un comentario. **Una arista significa exactamente que ese autor comentó ese video**, y nada más; no implica amistad, conversación ni acuerdo con el contenido.

| Variable y descripción                                                        | Tipo y nivel          | Unidades    | Valores / rango / ejemplo                                            |
| ----------------------------------------------------------------------------- | --------------------- | ----------- | -------------------------------------------------------------------- |
| **`autor`** — `author_channel_id` del extremo autor                           | Texto · Identificador | —           | 332 valores distintos                                                |
| **`video`** — `video_id` del extremo video                                    | Texto · Identificador | —           | 19 valores distintos                                                 |
| **`peso`** — Número de comentarios de ese autor en ese video                  | Entero · De razón     | Comentarios | 1 a 6. 303 de las 343 aristas tienen peso 1. La suma de pesos es 406 |
| **`likes_sumados`** — "Me gusta" acumulados por esos comentarios              | Entero · De razón     | "Me gusta"  | 0 a 405                                                              |
| **`canal_del_video`** — Canal que publicó el video, como atributo de contexto | Texto · Nominal       | —           | 8 valores                                                            |


### 3.3 `aristas_autor_autor.csv`

Proyección autor–autor: dos autores se conectan si comentaron el mismo video.

> **Nota:**  
> Sus 10,732 aristas son en su mayoría un **artefacto de la proyección** (cada video con `k` autores genera automáticamente `k(k−1)/2` conexiones). El video más comentado aporta por sí solo 8,128 aristas (75.7%), y solo **2 pares de autores** comparten realmente más de un video. La densidad y el agrupamiento altos de esta red no evidencian cohesión social.

| Variable y descripción                                                                          | Tipo y nivel          | Unidades | Valores / rango / ejemplo           |
| ----------------------------------------------------------------------------------------------- | --------------------- | -------- | ----------------------------------- |
| **`autor_A`** — Primer autor del par                                                            | Texto · Identificador | —        | Ej.: `UC-HeUTT6_g-VoiWds2a4H-w`     |
| **`autor_B`** — Segundo autor del par. La arista es **no dirigida**: el orden no significa nada | Texto · Identificador | —        | Ej.: `UC3YpzAr7pcdKC7BVFEpYzog`     |
| **`videos_compartidos`** — Peso de la arista: videos en que ambos comentaron                    | Entero · De razón     | Videos   | 1 o 2. Solo 2 aristas tienen peso 2 |


### 3.4 `aristas_video_video.csv`

Proyección video–video: dos videos se conectan si comparten al menos un comentarista. Con 11 aristas y 9 de los 19 videos aislados, describe un ecosistema muy fragmentado.

| Variable y descripción                                                | Tipo y nivel        | Unidades | Valores / rango / ejemplo                         |
| --------------------------------------------------------------------- | ------------------- | -------- | ------------------------------------------------- |
| **`video_A`** — Título del primer video del par                       | Texto · Texto libre | —        | Ej.: `Internet: escoger el menos malo`            |
| **`canal_A`** — Canal del primer video                                | Texto · Nominal     | —        | 5 valores                                         |
| **`video_B`** — Título del segundo video. Arista **no dirigida**      | Texto · Texto libre | —        | Ej.: `Arroz con pollo a la MONOPOLIO`             |
| **`canal_B`** — Canal del segundo video                               | Texto · Nominal     | —        | `Quorum`, `Gobierno de la República de Guatemala` |
| **`autores_compartidos`** — Peso de la arista: comentaristas en común | Entero · De razón   | Autores  | 1 o 2                                             |
| **`mismo_canal`** — Si ambos videos pertenecen al mismo canal         | Lógico · Nominal    | —        | `True` en 7 de las 11 aristas                     |


### 3.5 `metricas_topologia.csv`

Una fila por red analizada.

| Variable y descripción                                                                                                                                                              | Tipo y nivel       | Unidades    | Valores / rango / ejemplo                                                   |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ | ----------- | --------------------------------------------------------------------------- |
| **`red`** — Red descrita en la fila                                                                                                                                                 | Texto · Nominal    | —           | `bipartita autor–video`, `proyección autor–autor`, `proyección video–video` |
| **`nodos`** — Número de nodos                                                                                                                                                       | Entero · De razón  | Nodos       | 19, 332, 351                                                                |
| **`aristas`** — Número de aristas                                                                                                                                                   | Entero · De razón  | Aristas     | 11, 343, 10,732                                                             |
| **`densidad`** — Proporción de aristas existentes sobre las posibles, fórmula general                                                                                               | Decimal · De razón | Proporción  | 0.0056 a 0.1953                                                             |
| **`densidad_bipartita`** — Densidad correcta para un grafo bipartito: solo cuenta como posibles las aristas entre los dos conjuntos. **Vacía en las proyecciones**, donde no aplica | Decimal · De razón | Proporción  | 0.0544                                                                      |
| **`grado_medio`** — Vecinos por nodo en promedio                                                                                                                                    | Decimal · De razón | Vecinos     | 1.16 a 64.65                                                                |
| **`componentes`** — Piezas separadas de la red                                                                                                                                      | Entero · De razón  | Componentes | 10 en las tres redes                                                        |
| **`gigante_nodos`** — Nodos de la componente más grande                                                                                                                             | Entero · De razón  | Nodos       | 10 a 286                                                                    |
| **`gigante_%`** — Porcentaje de la red que ocupa la componente mayor                                                                                                                | Decimal · De razón | Porcentaje  | 52.6 a 83.1                                                                 |
| **`aislados`** — Nodos sin ninguna arista                                                                                                                                           | Entero · De razón  | Nodos       | 0 a 9                                                                       |
| **`transitividad`** — Proporción de triángulos cerrados. **Es 0 por construcción en la red bipartita**, que no puede tener triángulos                                               | Decimal · De razón | Proporción  | 0 a 0.984                                                                   |
| **`clustering_medio`** — Coeficiente de agrupamiento promedio                                                                                                                       | Decimal · De razón | Proporción  | 0 a 0.972                                                                   |
| **`diametro_gigante`** — Distancia máxima entre dos nodos de la componente mayor                                                                                                    | Entero · De razón  | Aristas     | 5 a 12                                                                      |
| **`camino_medio_gigante`** — Distancia media entre nodos de la componente mayor                                                                                                     | Decimal · De razón | Aristas     | 2.36 a 4.72                                                                 |


### 3.6 `comunidades_nodos.csv`

**Hereda las 15 variables** de `nodos_bipartita.csv` y añade una. Producido por el algoritmo de Louvain sobre la red bipartita ponderada (modularidad Q = 0.777).

| Variable y descripción                                                                                                                                                                     | Tipo y nivel     | Unidades | Valores / rango / ejemplo                                                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------- | -------- | -------------------------------------------------------------------------------------------- |
| **`comunidad`** — Comunidad asignada al nodo, numerada de mayor a menor tamaño. **Las etiquetas no tienen orden**: la comunidad 0 no es "mayor" en ningún sentido salvo el número de nodos | Entero · Nominal | —        | 0 a 16 (17 comunidades). Tamaños: 126, 48, 32, 31, 26, 19, 19, 14, 8, 8, 5, 4, 3, 2, 2, 2, 2 |


### 3.7 `centralidades.csv`

Una fila por nodo. Las medidas basadas en caminos **solo se definen dentro de una componente conexa**, así que se calcularon sobre la componente gigante (286 nodos) y quedan **vacías** en los 65 restantes. Un valor vacío significa "la medida no existe para este nodo", **no** "vale cero".

| Variable y descripción                                                                                                                                             | Tipo y nivel          | Unidades            | Valores / rango / ejemplo                                   |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------- | ------------------- | ----------------------------------------------------------- |
| **`node_id`** — Identificador del nodo                                                                                                                             | Texto · Identificador | —                   | 351 valores                                                 |
| **`grado_bip`** — Centralidad de grado normalizada para redes bipartitas: fracción del **otro** conjunto con la que el nodo se conecta. Mide participación directa | Decimal · De razón    | Proporción          | 0.0030 a 0.3855                                             |
| **`interm_bip`** — Intermediación: fracción de caminos más cortos que pasan por el nodo. Identifica **puentes** entre regiones de la red                           | Decimal · De razón    | Proporción          | 0 a 0.8539. **65 valores vacíos**. Solo 9 autores superan 0 |
| **`cercania_bip`** — Cercanía: inverso de la distancia media al resto. Mide rapidez para alcanzar la red                                                           | Decimal · De razón    | Proporción          | 0.1482 a 0.6407. **65 valores vacíos**                      |
| **`pagerank`** — PageRank ponderado: prestigio con amortiguación                                                                                                   | Decimal · De razón    | Proporción (suma 1) | 0.0012 a 0.1673                                             |
| **`vector_propio`** — Centralidad de vector propio: importancia por estar conectado a nodos importantes                                                            | Decimal · De razón    | Proporción          | 1.1e-08 a 0.707. **65 valores vacíos**                      |
| **`tipo`** — Conjunto del nodo                                                                                                                                     | Texto · Nominal       | —                   | `autor`, `video`                                            |
| **`grado`** — Grado sin normalizar, para comparar con `grado_bip`                                                                                                  | Entero · De razón     | Vecinos             | 1 a 128                                                     |
| **`etiqueta`** — Nombre visible para gráficos                                                                                                                      | Texto · Texto libre   | —                   | Ej.: `@josueramirez-ey3ih`                                  |


## 4. Resultados de sentimiento (`data/processed/`)

### 4.1 `comentarios_sentimiento.parquet`

Producido por el Notebook 4. **Hereda** `comment_id`, `video_id`, `author_channel_id`, `texto_original_comentario`, `channel_name`, `category`, `like_count` y `reply_count`.

| Variable y descripción                                                                                                                                                                                                  | Tipo y nivel       | Unidades       | Valores / rango / ejemplo                                                           |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ | -------------- | ----------------------------------------------------------------------------------- |
| **`sentimiento`** — Clase asignada por el modelo RoBERTuito, entrenado con tuits en español. Se aplicó sobre `texto_original`, no sobre el texto limpio, porque necesita puntuación, mayúsculas y emojis                | Texto · Nominal    | —              | `NEG` (250), `POS` (82), `NEU` (74)                                                 |
| **`sentimiento_prob`** — Probabilidad que el modelo asigna a la clase elegida. **Debe consultarse antes de tratar la etiqueta como certeza**                                                                            | Decimal · De razón | Probabilidad   | 0.355 a 0.986. Mediana 0.883. El 15.5% está por debajo de 0.60                      |
| **`lexico_score`** — Polaridad media de las palabras del comentario según ML-SentiCon. **Se conserva solo como referencia**: este enfoque se descartó por confundir sistemáticamente comentarios críticos con positivos | Decimal · De razón | Escala −1 a +1 | −0.875 a 0.813. **155 valores vacíos** (comentarios sin ninguna palabra del léxico) |
| **`lexico_etiqueta`** — `lexico_score` discretizado en tres clases, con corte en ±0.05. Coincide con `sentimiento` en solo el 52.6% de los casos                                                                        | Texto · Nominal    | —              | `NEG`, `NEU`, `POS`, `nan`                                                          |
| **`comunidad`** — Comunidad del autor del comentario, según el Ejercicio 7. Permite comparar el sentimiento entre comunidades                                                                                           | Entero · Nominal   | —              | 0 a 16                                                                              |
