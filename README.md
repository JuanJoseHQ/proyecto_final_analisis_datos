# Proyecto Integrador — Airbnb Ciudad de México

Análisis de los anuncios de Airbnb en Ciudad de México (Inside Airbnb, snapshot del 25 de junio al
2 de julio de 2025): ETL, análisis exploratorio, modelo dimensional con dashboard en Power BI y
modelo predictivo del precio por noche.

**Equipo:** Santiago Rodríguez · Juan José Herrera · Jean Carlos González

## Pipeline

| # | Notebook | Fase | Entrada → salida |
|---|---|---|---|
| 1 | `listings_clean.ipynb` | ETL | `bronze/listings.csv` (26,401 × 77) → `silver/listings.csv` (26,329 × 57) |
| 2 | `02_eda_listings.ipynb` | EDA | silver → `gold/listings_encoded.csv` (26,322 × 67) |
| 3 | `03_modelo_estrella.ipynb` | BI | silver → `gold/star/` (1 tabla de hechos + 4 dimensiones) |
| 4 | `03_modelo_supervisado.ipynb` | Modelo predictivo | gold → métricas y figuras del modelo de precio |

## Resultados

- **ETL:** se eliminó solo el 0.3% de los registros (precios y capacidades imposibles).
- **EDA:** el precio depende sobre todo del tipo de alojamiento, la ubicación y el tamaño; tres
  alcaldías concentran el 74% de la oferta.
- **BI:** modelo en estrella y dashboard de una página que responde tres preguntas de negocio:
  dónde se concentra la oferta, qué tipo de alojamiento rinde más y si ser superhost da ventaja.
- **Modelo:** HistGradientBoosting sin PCA, evaluado en anfitriones no vistos: R² = 0.711 (log10 del
  precio) y error porcentual mediano de 22.7%, frente a 47.3% de la línea base.

## Estructura

```
data/
  bronze/listings.csv          # dato crudo (versionado)
  silver/, gold/, gold/star/   # generados por los notebooks (no versionados)
notebook/                      # los 4 notebooks del pipeline
reports/
  figures/                     # figuras del ETL, eda/ y modelo/
  powerbi/Dashboard_Listings.pbix
  Informe_Proyecto_Integrador.docx
  PI_AD_2026-2.pdf             # guía del proyecto
```

## Cómo ejecutar

Requiere Python 3.12 o superior:

```bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn jupyter
```

Ejecuta los notebooks **desde la carpeta `notebook/`** (usan rutas relativas `../data/...`) y en el
orden de la tabla. El EDA y el modelo supervisado generan por sí mismos las capas anteriores si no
existen; el modelo supervisado tarda unos 10 minutos.

El dashboard (`reports/powerbi/Dashboard_Listings.pbix`) incluye los datos cargados y se abre
directamente en Power BI Desktop. Para actualizarlo, ejecuta primero `03_modelo_estrella.ipynb`,
ajusta el parámetro `RutaDatos` (**Transformar datos → Editar parámetros**) a tu carpeta
`data/gold/star/` y usa **Inicio → Actualizar**.
