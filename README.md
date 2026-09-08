# Proyecto Final — Análisis de Datos

Pipeline de **ingesta, limpieza y transformación** (arquitectura *bronze → silver*) sobre el
conjunto de datos de anuncios de **Airbnb en Ciudad de México**, desarrollado como proyecto final
del curso de Análisis de Datos.

## Equipo

- Santiago Rodríguez
- Juan José Herrera
- Jean Carlos

## Objetivo

Aplicar el proceso completo de análisis de datos —ingesta, diagnóstico, limpieza, transformación y
exploración— sobre el conjunto de datos seleccionado por el equipo, dejando cada decisión de
limpieza respaldada por la evidencia que la justifica.

## Conjunto de datos

| | |
|---|---|
| **Fuente** | Listings de Airbnb — Ciudad de México |
| **Captura** | Scrape del 25 de junio al 2 de julio de 2025 |
| **Volumen crudo** | 26,401 anuncios × 77 columnas |
| **Resultado curado** | 26,329 anuncios × 57 columnas |

## Estructura del repositorio

```
.
├── data/
│   ├── bronze/              # Datos crudos, tal como se ingieren desde la fuente
│   │   └── listings.csv
│   └── silver/              # Datos limpios y estandarizados (generado, no versionado)
│       └── listings.csv
├── notebook/
│   ├── listings.ipynb       # Exploración inicial del conjunto de datos
│   └── listings_clean.ipynb # Pipeline ETL principal: diagnóstico + transformaciones
└── reports/
    └── figures/             # Gráficas exportadas por el pipeline (10 PNG)
```

### Capas de datos

| Capa   | Descripción                                                                 |
|--------|-----------------------------------------------------------------------------|
| Bronze | Datos en bruto sin procesar, conservados en su formato original.            |
| Silver | Datos depurados: sin duplicados, con tipos corregidos y esquema unificado.  |

> `data/silver/listings.csv` no se versiona: es salida generada y se reconstruye ejecutando
> `listings_clean.ipynb` de principio a fin.

## Notebooks

### `notebook/listings_clean.ipynb` — pipeline principal

Se ejecuta de arriba a abajo y está dividido en dos mitades: primero **diagnostica el dato crudo**
y después aplica las **transformaciones**, cada una acompañada de la evidencia que la sustenta.

| Sección | Contenido |
|---|---|
| 1. Ingesta | Lectura del bronze sin conversiones implícitas |
| 2. Diagnóstico | Duplicados, valores faltantes, inconsistencias tipográficas, formato de `price`, distribuciones |
| 3. Transformaciones | Deduplicación, corrección de `price`, tasas con `%`, categorización de `host_response_time`, binarias de `host_verifications` y `amenities`, normalización de `bathrooms` y texto, tipado booleano, imputación y outliers |
| 4. Subset curado | Selección de las 57 columnas con valor analítico |
| 5. Validaciones | Unicidad de `id`, rango de precios, ausencia de listas sin serializar, nulos imputados |
| 6. Gráficas derivadas | Figuras que solo son posibles después de la limpieza |
| 7. Persistencia | Escritura en la capa silver y relectura de verificación |

**Decisiones de limpieza destacadas**

- **Sin duplicados:** verificado en tres niveles (fila completa, sin `_id` de Mongo, y por `id` de negocio).
- **Nulos:** se descartan las columnas de texto libre con más del 45% de faltantes. Solo se imputa
  `reviews_per_month` a `0`, donde el faltante tiene un significado conocido de negocio (3,373 registros).
- **`price`:** se retiran `$` y separador de miles conservando el decimal; mediana de **$1,039 MXN/noche**.
- **Outliers:** se descartan únicamente los valores imposibles de negocio — el 0.3% de los registros.
- **Amenities:** de 6,513 servicios distintos se derivan los 10 más comunes como columnas binarias,
  más un conteo total de equipamiento por anuncio.

### `notebook/listings.ipynb` — exploración inicial

Recorrido exploratorio previo sobre el dataset crudo: análisis de columnas `object` y numéricas,
valores nulos, `describe()`, histogramas y boxplots. Sirve como bitácora del análisis que motivó
las decisiones implementadas en el pipeline.

## Figuras generadas

Exportadas a `reports/figures/` al ejecutar `listings_clean.ipynb`:

| Archivo | Contenido |
|---|---|
| `01_nulos_pct_por_columna.png` | Porcentaje de nulos por columna |
| `02_heatmap_nulos.png` | Patrón de valores faltantes |
| `03_histogramas_numericas.png` | Distribución de las variables numéricas |
| `04_boxplots_numericas_crudo.png` | Valores atípicos en el dato crudo |
| `05_top_amenities.png` | Servicios más frecuentes |
| `06_boxplot_price_pre_recorte.png` | `price` antes del tratamiento de outliers |
| `07_price_post_limpieza.png` | `price` tras la limpieza (escala logarítmica) |
| `08_price_por_room_type.png` | Precio por tipo de alojamiento |
| `09_host_response_category.png` | Distribución de la categoría de respuesta del anfitrión |
| `10_correlacion.png` | Matriz de correlación |

## Cómo ejecutar

**Requisitos:** Python 3.12 con `pandas`, `numpy`, `matplotlib`, `seaborn` y `jupyter`.

```bash
pip install pandas numpy matplotlib seaborn jupyter
```

Los notebooks usan rutas relativas (`../data/...`), por lo que **deben ejecutarse desde la carpeta
`notebook/`**:

```bash
cd notebook
jupyter notebook listings_clean.ipynb
```

Ejecuta el notebook completo de arriba a abajo. Al finalizar se generan `data/silver/listings.csv`
y las 10 figuras en `reports/figures/`.
