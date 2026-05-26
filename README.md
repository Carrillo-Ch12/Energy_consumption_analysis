# Análisis empírico MongoDB vs. Apache Hive — Diseño físico, rendimiento y consumo energético

Notebook de análisis de datos del estudio experimental que compara **MongoDB 8.0.17** (NoSQL documental) y **Apache Hive 3.1.3** (data warehouse sobre Hadoop) bajo el benchmark **TPC-H (SF = 5 GB)**, midiendo **tiempo de ejecución** y **consumo energético** (vía Scaphandre / Intel RAPL) para cuatro configuraciones de diseño físico (*baseline*, índices, compresión Snappy e índices + compresión) en modalidades de despliegue **distribuida** y **centralizada**.

Este repositorio contiene el cuaderno (`Mongo_Hive.ipynb`) que carga las métricas crudas, las depura, calcula estadísticos y pruebas inferenciales, y genera todas las figuras del informe de tesis.

## Qué hace el notebook

A partir de las métricas recolectadas en los experimentos (CSV de potencia y tiempo por consulta, configuración, iteración y nodo), el cuaderno:

- Carga y valida los datos (QA), descartando archivos incompletos.
- Calcula la energía por **integración trapezoidal** sobre la serie de potencia.
- Aplica el **cap proporcional a 3 horas** (10 800 s) para tratar los *timeouts* como observaciones censuradas.
- Agrega en dos niveles: promedio por consulta (Q1–Q22) y mediana entre consultas por configuración.
- Ejecuta **correlación de Spearman** (tiempo–energía) y **test de Wilcoxon pareado** con corrección **Holm–Bonferroni** (α = 0,05).
- Construye el **ranking de configuraciones** por victorias/derrotas significativas.
- Genera las visualizaciones: barras por consulta (escala log), comparativos distribuido vs. centralizado, dispersión tiempo–energía, distribución de carga por nodo/shard, mapas de calor y gráficos radar.

## Estructura del cuaderno

1. **Setup** — clona automáticamente los 4 repositorios de datos.
2. **Imports y configuración global** — librerías y constantes (timeout, escenarios, patrón de CSV).
3. **Funciones auxiliares** — helpers para Hive (3.1) y MongoDB (3.2).
4. **Análisis Hive** — carga y QA (4.1); estadísticas y tablas (4.2).
5. **Análisis Mongo** — carga y QA (5.1); estadísticas y tablas (5.2).
6. **Gráficos Hive**.
7. **Gráficos Mongo**.
8. **Test de Wilcoxon — Hive**.
9. **Test de Wilcoxon — Mongo**.
10. **Radar charts** — comparación multidimensional (5 ejes normalizados).
11. **Radar por consulta** — tiempo de ejecución en 22 ejes (variantes 11.b–11.d).

Convención de sufijos: `_H` = Hive · `_M` = MongoDB.

## Datos

La primera celda (**Setup**) clona los cuatro repositorios con las métricas crudas en la carpeta del notebook, así que no es necesario descargarlos a mano:

| Motor | Modalidad | Repositorio |
|---|---|---|
| Apache Hive | Distribuido  | https://github.com/Carrillo-Ch12/Hive_metricas_new_5G |
| Apache Hive | Centralizado | https://github.com/Carrillo-Ch12/5g_hive_centralizado |
| MongoDB     | Distribuido  | https://github.com/Carrillo-Ch12/Metricas_5g |
| MongoDB     | Centralizado | https://github.com/Carrillo-Ch12/Mongo_centralizado_5G |

Cada CSV contiene, entre otras, las columnas `row_type`, `iteration`, `elapsed_seconds` y `power_total_watts`.

## Requisitos

- Python 3.10 o superior
- `git` disponible en el `PATH` (lo usa la celda de Setup)
- Paquetes: `numpy`, `pandas`, `matplotlib`, `seaborn`, `scipy`, `jupyter`

Instalación rápida:

```bash
pip install numpy pandas matplotlib seaborn scipy jupyter
```

## Cómo ejecutar

```bash
git clone <URL-de-este-repositorio>
cd Mongo-Hive-Analisis
pip install numpy pandas matplotlib seaborn scipy jupyter
jupyter notebook Mongo_Hive.ipynb
```

Ejecuta las celdas en orden (la celda de Setup clona los datos en la primera corrida). El notebook se publica **sin salidas** para mantenerlo liviano; las figuras y tablas se regeneran al ejecutarlo de principio a fin.

## Salidas generadas

Al ejecutarse, el cuaderno produce las figuras usadas en el informe, entre ellas los gráficos de barras de tiempo y energía por consulta, los comparativos distribuido vs. centralizado, los diagramas de dispersión tiempo–energía, los mapas de calor y los gráficos radar (multidimensional y por consulta).

## Relación con la tesis

Este cuaderno es el soporte reproducible del capítulo de **Resultados** del trabajo *"Estudio Empírico del Diseño Físico de Bases de Datos y su Impacto en la Eficiencia y el Consumo Energético"*. Las medianas, rankings de Wilcoxon y correlaciones de Spearman reportados en el informe se obtienen directamente de la ejecución de este notebook.
