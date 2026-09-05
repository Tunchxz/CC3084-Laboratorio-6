# Laboratorio 6: Análisis de Redes Sociales en YouTube

Este laboratorio estudia la **estructura de participación** de los usuarios en una muestra de videos de YouTube de canales guatemaltecos. A partir de dos conjuntos de datos recolectados por scraping (293 videos y 406 comentarios) se analiza quién participa, cómo se conectan los contenidos entre sí a través de sus audiencias, qué se dice en los comentarios y con qué carga emocional.

El trabajo combina dos enfoques complementarios:

- **Análisis de redes**: se modela la participación como una red bipartita autor–video, se construyen sus proyecciones, se mide su topología, se detectan comunidades y se identifican los nodos centrales y los participantes puente.
- **Análisis de contenido**: se limpia y lematiza el texto en español, se estudian palabras, bigramas y hashtags, y se clasifica el sentimiento de cada comentario.

## Estructura del repositorio

```
CC3084-Laboratorio-6/
├── notebooks/          # los 10 ejercicios en cuatro notebooks separados (en orden)
├── src/utils.py        # rutas del proyecto, estilo de gráficos y ayudas de lectura/escritura
├── data/raw/           # datos de entrada, no se modifican
├── data/processed/     # datos generados por los notebooks (no se versionan)
├── codebook.md         # diccionario de datos: todas las variables, crudas y derivadas
├── Informe.pdf         # informe de resultados, interpretación y conclusiones
└── requirements.txt    # dependencias
```

## Cómo ejecutar

### Requisitos

- **Python 3.12** (se usa [`uv`](https://docs.astral.sh/uv/) para crear el entorno)
- Conexión a internet en la primera ejecución, para descargar el modelo de `spaCy` y el de sentimiento

### 1. Colocar los datos

Los tres archivos de entrada deben estar en `data/raw/`:

```
data/raw/youtube_videos.csv
data/raw/youtube_comments.csv
```

### 2. Crear el entorno e instalar dependencias

```bash
uv venv --python 3.12 .venv

# torch debe instalarse desde el índice CPU de PyTorch.
# El wheel de PyPI incluye CUDA y pesa más del doble sin aportar nada en este proyecto.
VIRTUAL_ENV=.venv uv pip install torch --index-url https://download.pytorch.org/whl/cpu

VIRTUAL_ENV=.venv uv pip install -r requirements.txt
```

### 3. Registrar el kernel de Jupyter

```bash
.venv/bin/python -m ipykernel install --user --name lab6 --display-name "Lab6 (py3.12)"
```

### 4. Ejecutar los notebooks **en orden**

El orden es obligatorio: cada notebook consume los archivos que produjo el anterior en `data/processed/`.

```bash
cd notebooks
for nb in 01_carga_y_limpieza 02_eda 03_redes 04_sentimiento_conclusiones; do
    ../.venv/bin/jupyter nbconvert --to notebook --execute --inplace $nb.ipynb
done
```

También pueden abrirse de forma interactiva, seleccionando el kernel **Lab6 (py3.12)**:

```bash
.venv/bin/jupyter lab
```

## Documentación adicional

- [**Codebook**](./codebook.md): descripción de cada variable de los datos crudos y de los generados, con su tipo, nivel de medición, unidades y valores permitidos.
- [**Informe**](./Informe.pdf): resultados, interpretación y conclusiones.
