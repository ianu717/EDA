# 🚦 UK Road Safety Analysis — STATS19

Análisis exploratorio de accidentalidad vial en el Reino Unido a partir de los datos oficiales del **Department for Transport (DfT)**, sistema de registro **STATS19**. El proyecto cubre los últimos 5 años disponibles (~2020–2024) y evalúa tres hipótesis sobre física del impacto, patrones temporales y geografía de los accidentes.

---

## Estructura del proyecto

```
EDA/
├── notebook/
│   ├── uk_road_safety_cleaning.ipynb    # Limpieza y preprocesado
│   └── uk_road_safety_analysis.ipynb    # EDA e hipótesis
├── src/
│   └── uk_data_mapper.py                # Decodificador de variables categóricas
├── data/
│   ├── raw/                             # CSVs originales del DfT
│   └── processed/                       # Parquets limpios para el análisis
└── report/                              # Gráficos y mapas exportados
```

---

## Fuentes de datos

Todos los datos son públicos y oficiales, publicados por el **DfT (GOV.UK)**:

| Dataset | URL |
|---|---|
| Colisiones (últimos 5 años) | [dft-road-casualty-statistics-collision-last-5-years.csv](https://data.dft.gov.uk/road-accidents-safety-data/dft-road-casualty-statistics-collision-last-5-years.csv) |
| Víctimas (últimos 5 años) | [dft-road-casualty-statistics-casualty-last-5-years.csv](https://data.dft.gov.uk/road-accidents-safety-data/dft-road-casualty-statistics-casualty-last-5-years.csv) |
| Vehículos (últimos 5 años) | [dft-road-casualty-statistics-vehicle-last-5-years.csv](https://data.dft.gov.uk/road-accidents-safety-data/dft-road-casualty-statistics-vehicle-last-5-years.csv) |
| Guía de codificación (2024) | [data-guide-2024.xlsx](https://assets.publishing.service.gov.uk/media/691c6440e39a085bda43eed6/dft-road-casualty-statistics-road-safety-open-dataset-data-guide-2024.xlsx) |
| Manual STATS20 (p. 55) | [stats20-2005.pdf](https://assets.publishing.service.gov.uk/government/uploads/system/uploads/attachment_data/file/995424/stats20-2005.pdf) |

> **Nota:** Las variables categóricas están codificadas como enteros. La función `create_data_map` en `src/uk_data_mapper.py` las traduce a texto a partir del Excel oficial.

---

## Volumen de datos

| Dataset | Filas aprox. |
|---|---|
| Colisiones | ~600.000 |
| Víctimas | ~1.000.000 |
| Vehículos | ~900.000 |

---

## Hipótesis evaluadas

### H1 — El tipo de impacto influye en la tasa de mortalidad

Se combinaron los datasets de colisiones y vehículos (`merge` por `collision_index`) y se calculó la **tasa de fatalidad** (% de colisiones fatales) por tipo de impacto: frontal, trasero, lateral y sin impacto directo.

**Hallazgo:** Los accidentes clasificados como *"Did not impact"* (el vehículo esquiva sin colisionar) presentan la **mayor tasa de mortalidad** pese a representar solo el 5% del total. La hipótesis explicativa es que la maniobra evasiva provoca la salida de la calzada, lo cual resulta igual o más letal que un impacto directo.

---

### H2 — Distribución mensual de accidentes repetitiva entre años

Se extrajo la componente temporal de `date_time` y se agrupó por año–mes para comparar patrones estacionales. Se usó un heatmap de correlaciones de Pearson entre años para cuantificar la similitud.

**Hallazgo:** Los años 2022–2024 presentan patrones muy similares (correlación de hasta **0,94** entre 2023 y 2024). Los años 2020 y 2021 rompen el patrón por efecto de los **confinamientos COVID-19** (marzo 2020 y enero 2021), claramente visibles en el gráfico de líneas.

---

### H3 — Los accidentes fatales se concentran en ciertas localidades

Se filtraron las colisiones con `collision_severity == 1` (fatal) y se utilizó **Folium** para generar mapas interactivos en HTML. Se identificaron además las 5 carreteras con mayor número de siniestros mortales.

**Hallazgo:** La mayor concentración se da en las áreas metropolitanas (Londres, Manchester, Birmingham). Sin embargo, varias de las carreteras son autopistas interurbanas de larga distancia donde la alta velocidad eleva la gravedad.

---

## Pipeline de ejecución

```
1. uk_road_safety_cleaning.ipynb
   ├── Carga de CSVs desde URLs del DfT
   ├── Inspección: nulos, duplicados, valores -1
   ├── Creación de columna date_time (date + time → datetime)
   ├── Eliminación de columnas irrelevantes
   └── Exportación → data/processed/*.parquet

2. uk_road_safety_analysis.ipynb
   ├── Carga de parquets procesados
   ├── Decodificación categórica con create_data_map
   ├── H1: merge collision + vehicle → tasa de fatalidad por impacto
   ├── H2: extracción temporal → correlación mensual entre años
   └── H3: filtrado fatal → mapas Folium + top-5 carreteras
```

---

## Tecnologías utilizadas

| Librería | Uso |
|---|---|
| `pandas` | Carga, limpieza, merges y agregaciones |
| `numpy` | Recategorización condicional (`np.select`) |
| `matplotlib` | Gráficos base y exportación PNG |
| `seaborn` | Barplots, lineplot y heatmap de correlaciones |
| `folium` | Mapas interactivos HTML (HeatMap + CircleMarker) |
| `pathlib` | Gestión de rutas |

---

## Decisiones técnicas destacadas

- **Formato Parquet** para los datos procesados: mejor compresión y velocidad de lectura que CSV a este volumen.
- **Valores -1 como "desconocido"**: se filtran específicamente en cada hipótesis en lugar de eliminarse globalmente, evitando pérdida innecesaria de datos.
- **Recategorización de impactos laterales**: `Nearside` y `Offside` se unifican en `Side` para simplificar sin perder información.
- **Locale español** (`es_ES.UTF-8`) para los nombres de meses en el análisis estacional.

---

## Notas

- Los datos de 2020–2021 deben interpretarse con cautela por el efecto de los confinamientos.
- La columna `generic_make_model` del dataset de vehículos presenta tipos mixtos; se fuerza `dtype=str` en la carga.
- Los mapas Folium se generan como archivos `.html` independientes y requieren un navegador para visualizarse de forma interactiva.
