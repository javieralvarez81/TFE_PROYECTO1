# Predicción del riesgo de rotura de stock

## Proyecto de Machine Learning aplicado a un entorno logístico

Este repositorio contiene un proyecto completo de **Machine Learning** cuyo objetivo es identificar, con antelación, qué combinaciones de **tienda y producto** presentan un mayor riesgo de sufrir una rotura de stock.

Una rotura de stock ocurre cuando un producto no está disponible en el momento en que existe demanda. Esto puede provocar pérdida de ventas, menor nivel de servicio, pedidos urgentes, replanificaciones y una peor experiencia para el cliente.

El proyecto utiliza datos históricos de ventas, precios, promociones y comportamiento temporal para construir un sistema de apoyo a la decisión. El modelo no sustituye a un sistema de gestión de almacén ni decide automáticamente cuánto comprar. Su función es **priorizar los casos que deberían ser revisados primero por un equipo de planificación o aprovisionamiento**.

---

## Índice

1. [Resumen para una persona no técnica](#resumen-para-una-persona-no-técnica)
2. [Objetivo de negocio](#objetivo-de-negocio)
3. [Qué pregunta responde el proyecto](#qué-pregunta-responde-el-proyecto)
4. [Alcance y limitación principal](#alcance-y-limitación-principal)
5. [Cómo funciona el proyecto](#cómo-funciona-el-proyecto)
6. [Esquema completo del sistema](#esquema-completo-del-sistema)
7. [Los cuatro notebooks](#los-cuatro-notebooks)
8. [Estructura del repositorio](#estructura-del-repositorio)
9. [Descripción de los datos](#descripción-de-los-datos)
10. [Variables utilizadas](#variables-utilizadas)
11. [Modelos comparados](#modelos-comparados)
12. [Métricas de evaluación](#métricas-de-evaluación)
13. [Explicabilidad y análisis de errores](#explicabilidad-y-análisis-de-errores)
14. [Traducción del modelo a negocio](#traducción-del-modelo-a-negocio)
15. [Cómo ejecutar el proyecto](#cómo-ejecutar-el-proyecto)
16. [Archivos generados](#archivos-generados)
17. [Cómo podría utilizarse en una empresa](#cómo-podría-utilizarse-en-una-empresa)
18. [Propuesta conceptual de puesta en producción](#propuesta-conceptual-de-puesta-en-producción)
19. [Limitaciones](#limitaciones)
20. [Mejoras futuras](#mejoras-futuras)
21. [Tecnologías utilizadas](#tecnologías-utilizadas)

---

## Resumen para una persona no técnica

El funcionamiento general puede entenderse con este ejemplo:

1. El sistema revisa el historial de ventas de cada producto en cada tienda.
2. Observa si las ventas recientes están subiendo o bajando.
3. Tiene en cuenta aspectos como el precio, las promociones, el día de la semana y el plazo de reposición.
4. Aprende de ejemplos históricos previamente preparados.
5. Para cada combinación tienda-producto, genera una probabilidad de riesgo.
6. Los casos con mayor probabilidad se sitúan al principio de una lista.
7. El equipo de negocio puede revisar primero los casos más urgentes.

El resultado no debe interpretarse como:

> “Este producto se quedará sin stock con total seguridad”.

La interpretación correcta es:

> “Según los patrones aprendidos, este caso presenta más riesgo que otros y debería revisarse antes”.

Esta diferencia es importante porque un modelo predictivo no elimina la incertidumbre. Su utilidad consiste en ayudar a priorizar mejor el trabajo.

---

## Objetivo de negocio

El objetivo principal es construir un sistema capaz de:

- detectar patrones relacionados con un posible riesgo de falta de disponibilidad;
- asignar una probabilidad de riesgo a cada combinación de tienda y producto;
- ordenar los casos desde mayor hasta menor riesgo;
- ayudar a los equipos de planificación a concentrarse en los casos más importantes;
- reducir, de forma potencial, las ventas perdidas y las actuaciones urgentes;
- convertir los resultados técnicos del modelo en indicadores comprensibles para negocio.

El proyecto está planteado como un caso de uso logístico de clasificación binaria:

- `0`: situación clasificada como sin riesgo;
- `1`: situación clasificada como con riesgo.

---

## Qué pregunta responde el proyecto

La pregunta principal es:

> ¿Qué combinaciones de tienda y producto presentan mayor riesgo de no disponer de suficiente stock para cubrir la demanda esperada durante el plazo de reposición?

El modelo también permite responder preguntas secundarias:

- ¿Qué factores influyen más en la predicción?
- ¿Qué productos o tiendas acumulan más alertas?
- ¿Cuántos casos de riesgo consigue detectar?
- ¿Cuántas alertas incorrectas genera?
- ¿Qué umbral de decisión conviene utilizar?
- ¿Qué porcentaje de los casos de riesgo se detectaría revisando solo el 10 %, 20 % o 30 % con mayor probabilidad?
- ¿Cómo podría integrarse este sistema en un proceso logístico real?

---

## Alcance y limitación principal

El dataset original contiene información de ventas, precios y promociones, pero **no incluye inventario físico real, pedidos pendientes, fechas reales de reposición ni ventas perdidas**.

Por este motivo, no es posible observar directamente una rotura de stock real.

Para poder desarrollar el ejercicio, se construye un target denominado:

```text
stockout_risk
```

Este target representa una **aproximación sintética al riesgo**. Se calcula utilizando demanda histórica, demanda futura y variables logísticas simuladas de forma reproducible.

Por tanto:

- el proyecto sí permite evaluar un pipeline completo de Machine Learning;
- sí permite comparar modelos y estudiar patrones de riesgo;
- sí permite analizar cómo podría priorizarse la operación;
- no demuestra que las roturas detectadas hayan ocurrido realmente;
- las estimaciones económicas son escenarios simulados, no ahorros certificados.

En un proyecto empresarial real, el target debería construirse con información procedente del WMS, ERP o sistema de aprovisionamiento.

---

## Cómo funciona el proyecto

El repositorio sigue una secuencia de cuatro fases:

### Fase 1. Comprender los datos

Se analiza el dataset original para comprobar:

- qué columnas contiene;
- cuántos registros existen;
- qué periodo cubre;
- si hay valores nulos;
- si existen duplicados;
- cómo evolucionan las ventas;
- cómo se comportan tiendas y productos;
- qué relación aparente tienen el precio y las promociones con las ventas.

### Fase 2. Preparar la información para el modelo

Se crean variables que resumen el comportamiento reciente de cada producto:

- ventas de días anteriores;
- medias móviles;
- variabilidad de la demanda;
- tendencias;
- información del calendario;
- precio reciente;
- promociones recientes;
- plazo estimado de reposición.

En esta fase también se crea la variable objetivo `stockout_risk`.

### Fase 3. Entrenar y comparar modelos

Se dividen los datos respetando el orden temporal:

- los datos más antiguos se utilizan para entrenar;
- los datos más recientes se reservan para comprobar el comportamiento del modelo.

Después se comparan varios algoritmos y se selecciona el que ofrece el mejor equilibrio entre las métricas elegidas.

### Fase 4. Interpretar y convertir los resultados en decisiones

Se analiza:

- qué variables son más importantes;
- qué errores comete el modelo;
- cuántas alertas genera;
- qué ocurre al cambiar el umbral;
- qué porcentaje de riesgo puede capturarse revisando solo los casos prioritarios;
- cómo podría utilizarse el sistema dentro de una empresa.

---

## Esquema completo del sistema

```text
┌──────────────────────────────────────┐
│ Dataset original de ventas           │
│ retail_sales.csv                     │
│                                      │
│ Fecha · Tienda · Producto · Ventas   │
│ Precio · Promoción                   │
└───────────────────┬──────────────────┘
                    │
                    ▼
┌──────────────────────────────────────┐
│ Notebook 01                         │
│ Análisis exploratorio de datos       │
│                                      │
│ Revisión de calidad, distribución,   │
│ evolución temporal y patrones        │
└───────────────────┬──────────────────┘
                    │
                    ▼
┌──────────────────────────────────────┐
│ Notebook 02                         │
│ Ingeniería de variables              │
│                                      │
│ Lags, medias móviles, tendencias,    │
│ variables temporales y target        │
└───────────────────┬──────────────────┘
                    │
                    ▼
┌──────────────────────────────────────┐
│ Dataset preparado                    │
│ stockout_dataset.csv                 │
└───────────────────┬──────────────────┘
                    │
                    ▼
┌──────────────────────────────────────┐
│ Notebook 03                         │
│ Entrenamiento y evaluación           │
│                                      │
│ Baseline · Regresión logística       │
│ Random Forest · XGBoost              │
└───────────────────┬──────────────────┘
                    │
                    ├──────────────► Modelo seleccionado
                    ├──────────────► Métricas
                    ├──────────────► Predicciones
                    └──────────────► Gráficos
                    │
                    ▼
┌──────────────────────────────────────┐
│ Notebook 04                         │
│ Explicabilidad y negocio             │
│                                      │
│ Importancia de variables             │
│ Análisis de errores                  │
│ Umbrales y priorización              │
│ Escenarios económicos simulados      │
└───────────────────┬──────────────────┘
                    │
                    ▼
┌──────────────────────────────────────┐
│ Resultado para negocio               │
│                                      │
│ Lista priorizada de riesgos          │
│ Recomendación de umbral              │
│ Informes, métricas y visualizaciones │
└──────────────────────────────────────┘
```

---

## Los cuatro notebooks

Los notebooks deben ejecutarse en orden porque cada uno utiliza resultados generados por el anterior.

### 1. `01_eda_nivel_curso.ipynb`

Este notebook realiza el análisis exploratorio de datos, conocido como EDA.

Su finalidad es conocer el dataset antes de modificarlo o utilizarlo para entrenar modelos.

Incluye:

- carga del archivo original;
- revisión de nombres y tipos de columnas;
- conversión de fechas y valores numéricos;
- análisis de valores ausentes;
- comprobación de duplicados;
- estadísticas descriptivas;
- análisis temporal de las ventas;
- ventas por tienda y producto;
- comportamiento por día, mes y semana;
- análisis de precios y promociones;
- revisión de registros con ventas iguales a cero;
- generación de tablas y gráficos.

Una venta igual a cero no se considera automáticamente una rotura de stock. También puede significar que no hubo demanda. Por eso, los ceros se investigan, pero no se utilizan directamente como variable objetivo.

**Entradas principales:**

```text
data/raw/retail_sales.csv
```

**Salidas principales:**

```text
outputs/figures/eda/
outputs/tables/eda/
```

---

### 2. `02_feature_engineering_revisado.ipynb`

Este notebook transforma el dataset original en un conjunto de datos adecuado para Machine Learning.

La ingeniería de variables consiste en convertir datos básicos en indicadores que ayuden al modelo a encontrar patrones.

Por ejemplo, una fila original puede indicar que hoy se han vendido 20 unidades. El notebook añade información como:

- ventas de ayer;
- ventas de hace una semana;
- media de ventas de los últimos 7 días;
- media de ventas de los últimos 30 días;
- variabilidad reciente;
- tendencia de la demanda;
- número de días recientes en promoción;
- precio medio reciente;
- día de la semana;
- mes;
- semana del año;
- plazo estimado de reposición.

Este notebook también crea `stockout_risk`, la variable que el modelo intentará predecir.

Para evitar que el modelo utilice información del futuro, las variables históricas se calculan por separado para cada combinación de tienda y producto y se desplazan cuando corresponde.

**Entradas principales:**

```text
data/raw/retail_sales.csv
```

**Salidas principales:**

```text
data/processed/stockout_dataset.csv
outputs/tables/feature_engineering/feature_catalog.csv
outputs/tables/feature_engineering/feature_engineering_metadata.json
outputs/figures/feature_engineering/
```

---

### 3. `03_model_training_comentado.ipynb`

Este notebook entrena y compara los modelos predictivos.

Se utilizan los datos generados en el notebook anterior y se realiza una separación temporal. Esto significa que el modelo aprende con el pasado y se evalúa con un periodo posterior.

No se utiliza una división aleatoria convencional porque mezclar fechas antiguas y futuras podría producir una evaluación poco realista.

Los modelos comparados son:

- Dummy Classifier, utilizado como referencia mínima;
- Regresión Logística;
- Random Forest;
- XGBoost.

El notebook calcula métricas de clasificación, genera gráficos y guarda:

- todos los resultados de comparación;
- las predicciones del conjunto de test;
- la lista de variables utilizadas;
- los metadatos del entrenamiento;
- el mejor modelo entrenado.

**Entrada principal:**

```text
data/processed/stockout_dataset.csv
```

**Salidas principales:**

```text
models/best_stockout_model.pkl
models/model_features.json
models/model_metadata.json
outputs/metrics/
outputs/predictions/test_predictions_best_model.csv
outputs/figures/model_training/
```

---

### 4. `04_explainability_business_optimizado.ipynb`

Este notebook analiza el modelo final desde una perspectiva técnica y empresarial.

Incluye cuatro bloques:

#### Evaluación final

Recalcula las métricas sobre las predicciones guardadas por el notebook 03 y genera:

- matriz de confusión;
- curva ROC;
- curva Precision-Recall;
- informe de clasificación.

#### Explicabilidad

Analiza qué variables influyen más en las predicciones mediante:

- importancia nativa del modelo;
- importancia por permutación;
- SHAP, cuando la librería y el tipo de modelo lo permiten.

La importancia de una variable no demuestra causalidad. Solo indica cuánto depende el modelo de esa información para realizar sus predicciones.

#### Análisis de errores

Se estudian cuatro resultados posibles:

- verdadero positivo: detecta correctamente un riesgo;
- falso positivo: genera una alerta que no correspondía con riesgo;
- falso negativo: no detecta un caso de riesgo;
- verdadero negativo: descarta correctamente un caso sin riesgo.

Desde el punto de vista logístico, los falsos negativos son especialmente relevantes porque representan riesgos que el sistema no habría señalado.

#### Métricas de negocio

Se comparan distintos umbrales de decisión y diferentes capacidades operativas.

Por ejemplo:

- revisar solo el 10 % con más riesgo;
- revisar el 20 %;
- revisar el 30 %;
- estimar cuántos casos positivos se capturan en cada escenario.

También se crean simulaciones económicas. Estas simulaciones dependen de costes hipotéticos y deben sustituirse por cifras reales antes de tomar decisiones empresariales.

**Entradas principales:**

```text
data/processed/stockout_dataset.csv
models/best_stockout_model.pkl
models/model_features.json
models/model_metadata.json
outputs/predictions/test_predictions_best_model.csv
```

**Salidas principales:**

```text
outputs/metrics/
outputs/figures/explainability_business/
outputs/reports/
```

---

## Estructura del repositorio

```text
TFM_BIGSCHOOL/
│
├── README.md
├── requirements.txt
│
├── data/
│   ├── raw/
│   │   └── retail_sales.csv
│   │
│   └── processed/
│       └── stockout_dataset.csv
│
├── notebooks/
│   ├── 01_eda_nivel_curso.ipynb
│   ├── 02_feature_engineering_revisado.ipynb
│   ├── 03_model_training_comentado.ipynb
│   └── 04_explainability_business_optimizado.ipynb
│
├── models/
│   ├── best_stockout_model.pkl
│   ├── model_features.json
│   └── model_metadata.json
│
├── outputs/
│   ├── figures/
│   │   ├── eda/
│   │   ├── feature_engineering/
│   │   ├── model_training/
│   │   └── explainability_business/
│   │
│   ├── tables/
│   │   ├── eda/
│   │   └── feature_engineering/
│   │
│   ├── metrics/
│   ├── predictions/
│   │   └── test_predictions_best_model.csv
│   │
│   └── reports/
│
└── src/
```

### Explicación de cada carpeta

#### `data/raw`

Contiene los datos originales.

Estos archivos deben mantenerse sin modificar para conservar una copia de la fuente inicial.

#### `data/processed`

Contiene datos transformados y preparados para el entrenamiento.

El archivo principal es:

```text
stockout_dataset.csv
```

#### `notebooks`

Contiene el desarrollo del proyecto dividido en cuatro fases.

Los números iniciales indican el orden de ejecución.

#### `models`

Contiene el modelo seleccionado y la información necesaria para reutilizarlo.

El archivo `.pkl` almacena el modelo ya entrenado.

#### `outputs/figures`

Contiene los gráficos generados en cada fase.

Se separan por notebook para facilitar su localización.

#### `outputs/tables`

Contiene tablas de análisis, controles de calidad y catálogos de variables.

#### `outputs/metrics`

Contiene las métricas de entrenamiento, evaluación, umbrales, errores e importancia de variables.

#### `outputs/predictions`

Contiene las predicciones realizadas sobre el conjunto de test.

Cada fila incluye el resultado real, la clase predicha y la probabilidad estimada.

#### `outputs/reports`

Contiene informes complementarios, ejemplos de errores y resultados de explicabilidad.

#### `src`

Está reservada para funciones reutilizables si el proyecto se transforma posteriormente en una aplicación o pipeline automatizado.

---

## Descripción de los datos

El dataset utiliza una granularidad diaria por:

```text
fecha + tienda + producto
```

Las columnas originales principales son:

| Columna | Significado |
|---|---|
| `date` | Fecha de la observación |
| `store_id` | Identificador de la tienda |
| `item_id` | Identificador del producto |
| `sales` | Unidades vendidas |
| `price` | Precio del producto |
| `promo` | Indica si el producto estaba en promoción |

El proyecto no interpreta una fila de forma aislada. Cada observación se relaciona con el historial de ventas anterior de esa misma combinación tienda-producto.

---

## Variables utilizadas

Las variables creadas pueden agruparse en varias familias.

### Variables de calendario

Representan el momento en el que se realiza la observación:

- año;
- mes;
- semana;
- día de la semana;
- fin de semana;
- variables cíclicas del calendario.

Estas variables permiten detectar estacionalidades y patrones repetitivos.

### Historial de ventas

Incluyen las ventas observadas en momentos anteriores:

- `sales_lag_1`;
- `sales_lag_7`;
- `sales_lag_14`;
- `sales_lag_30`.

Por ejemplo, `sales_lag_7` representa las ventas de la misma tienda y producto siete días antes.

### Ventanas móviles

Resumen el comportamiento reciente:

- media de 7 días;
- media de 14 días;
- media de 30 días;
- desviación estándar;
- mínimos y máximos;
- tendencia reciente.

Estas variables ayudan a distinguir entre una demanda estable y una demanda que está aumentando o disminuyendo.

### Precio y promoción

Incluyen:

- precio actual;
- precio medio reciente;
- comparación del precio actual con su media;
- promoción actual;
- promoción del día anterior;
- número de días recientes en promoción.

### Variables logísticas

Incluyen un plazo de reposición aproximado:

```text
lead_time_days
```

Este dato representa el tiempo que podría tardar la reposición del producto.

En este proyecto se simula de forma reproducible porque no existe en el dataset original.

---

## Modelos comparados

### Dummy Classifier

Es un modelo de referencia.

No intenta aprender relaciones complejas. Sirve para comprobar si los modelos reales aportan valor frente a una estrategia básica.

### Regresión Logística

Es un modelo lineal sencillo e interpretable.

Permite establecer una base sólida y entender si existe una relación aproximadamente lineal entre las variables y la probabilidad de riesgo.

### Random Forest

Combina múltiples árboles de decisión.

Puede detectar relaciones no lineales e interacciones entre variables, aunque consume más recursos y puede ser más difícil de interpretar.

### XGBoost

Es un modelo de boosting basado en árboles.

Construye árboles de forma secuencial, haciendo que cada nuevo árbol intente corregir errores de los anteriores.

En las pruebas realizadas fue el modelo con mejor rendimiento global, aunque la diferencia frente a otras alternativas no es extrema.

---

## Métricas de evaluación

No existe una única métrica suficiente para evaluar un problema de clasificación.

### Accuracy

Indica el porcentaje total de predicciones correctas.

Puede resultar engañosa cuando una clase es mucho más frecuente que la otra.

### Precision

De todas las alertas generadas, indica qué proporción correspondía realmente a casos positivos.

Una precision baja implica muchas alertas incorrectas.

### Recall

De todos los casos positivos, indica qué porcentaje consigue detectar el modelo.

En este proyecto es especialmente importante porque un falso negativo representa un riesgo que no habría sido advertido.

### F1-score

Combina precision y recall en una única métrica.

Es útil cuando se busca un equilibrio entre detectar riesgos y no generar demasiadas falsas alertas.

### ROC-AUC

Mide la capacidad del modelo para ordenar casos positivos por encima de casos negativos utilizando todos los umbrales posibles.

### PR-AUC

Resume la relación entre precision y recall.

Es especialmente útil cuando interesa estudiar la clase positiva y cuando existe desbalance.

---

## Resultados generales

Los resultados obtenidos muestran una capacidad predictiva moderada.

El modelo seleccionado logra detectar una proporción elevada de los casos positivos, pero también genera falsas alertas.

Esto significa que el sistema puede ser útil para **priorizar revisiones**, pero no debería ejecutar decisiones automáticas sin validación humana.

En una ejecución previa del proyecto, XGBoost obtuvo aproximadamente:

| Métrica | Resultado aproximado |
|---|---:|
| Accuracy | 0,61 |
| Precision | 0,63 |
| Recall | 0,80 |
| F1-score | 0,71 |
| ROC-AUC | 0,63 |

Estos valores deben actualizarse con los CSV generados por la ejecución definitiva del notebook 03.

La lectura de negocio sería:

- el modelo consigue recuperar una parte importante de los casos de riesgo;
- varias alertas serán falsos positivos;
- el sistema funciona mejor como herramienta de priorización que como sistema de decisión automática;
- todavía existe margen de mejora antes de un uso real.

---

## Explicabilidad y análisis de errores

Un modelo no debe evaluarse únicamente por su puntuación.

También es necesario entender:

- qué variables utiliza;
- qué comportamientos aprende;
- dónde falla;
- si sus errores se concentran en determinadas tiendas, productos o periodos.

### Importancia global

La importancia global muestra qué variables influyen más en el conjunto de predicciones.

Se utilizan:

- importancia propia del algoritmo;
- importancia por permutación;
- SHAP, cuando es compatible.

### Explicación individual

SHAP puede ayudar a explicar por qué una predicción concreta tiene una probabilidad alta o baja.

Por ejemplo, una alerta podría estar impulsada por:

- crecimiento reciente de las ventas;
- promedio móvil elevado;
- plazo de reposición largo;
- promociones recientes;
- diferencias entre la venta actual y la histórica.

### Análisis de falsos negativos

Los falsos negativos son casos de riesgo que el modelo no detecta.

Son importantes porque podrían traducirse en:

- falta de mercancía;
- pérdida de ventas;
- incumplimiento del nivel de servicio;
- expediciones urgentes.

### Análisis de falsos positivos

Los falsos positivos son alertas que finalmente no correspondían a un caso positivo.

Pueden provocar:

- revisiones innecesarias;
- exceso de stock;
- inmovilización de capital;
- pérdida de confianza en el sistema.

El objetivo no es eliminar completamente uno de los dos errores, sino encontrar un equilibrio razonable para la operación.

---

## Traducción del modelo a negocio

El modelo genera una probabilidad entre 0 y 1.

Por ejemplo:

```text
0,18 → riesgo bajo
0,52 → riesgo medio
0,87 → riesgo alto
```

Para convertir la probabilidad en una alerta se utiliza un umbral.

Con un umbral de `0,50`:

```text
probabilidad >= 0,50 → alerta
probabilidad < 0,50  → sin alerta
```

Sin embargo, el umbral de 0,50 no tiene por qué ser el mejor para la empresa.

### Umbral bajo

Genera más alertas:

- aumenta la capacidad de detectar riesgos;
- también incrementa los falsos positivos;
- requiere más capacidad de revisión.

### Umbral alto

Genera menos alertas:

- reduce el trabajo operativo;
- mejora normalmente la precisión;
- puede dejar riesgos sin detectar.

### Priorización por capacidad

Una empresa puede no tener capacidad para revisar todos los casos.

Una estrategia práctica sería revisar:

- el 10 % con mayor riesgo;
- el 20 %;
- el 30 %.

El notebook 04 calcula cuántos casos positivos se capturan en cada escenario.

Esto permite adaptar el modelo a los recursos disponibles, en lugar de utilizar el mismo umbral para todas las operaciones.

---

## Cómo ejecutar el proyecto

### 1. Descargar o clonar el repositorio

```bash
git clone <URL_DEL_REPOSITORIO>
cd TFM_BIGSCHOOL
```

### 2. Crear un entorno virtual

En Windows:

```bash
python -m venv .venv
.venv\Scripts\activate
```

En Linux o macOS:

```bash
python -m venv .venv
source .venv/bin/activate
```

### 3. Instalar las dependencias

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### 4. Comprobar el dataset

El archivo original debe encontrarse en:

```text
data/raw/retail_sales.csv
```

### 5. Abrir Jupyter

```bash
jupyter notebook
```

También puede utilizarse JupyterLab:

```bash
jupyter lab
```

### 6. Ejecutar los notebooks en orden

```text
1. 01_eda_nivel_curso.ipynb
2. 02_feature_engineering_revisado.ipynb
3. 03_model_training_comentado.ipynb
4. 04_explainability_business_optimizado.ipynb
```

No debe empezarse por el notebook 03 o 04, porque dependen de archivos generados previamente.

---

## Archivos generados

Después de ejecutar el proyecto completo deberían existir, entre otros, los siguientes archivos:

```text
data/processed/stockout_dataset.csv

models/best_stockout_model.pkl
models/model_features.json
models/model_metadata.json

outputs/predictions/test_predictions_best_model.csv

outputs/metrics/model_comparison.csv
outputs/metrics/final_model_metrics.csv
outputs/metrics/threshold_analysis.csv
outputs/metrics/error_type_summary.csv
outputs/metrics/permutation_feature_importance.csv

outputs/reports/representative_prediction_errors.csv
outputs/reports/shap_execution_status.json
```

Los nombres exactos pueden variar ligeramente según la versión del notebook, pero la organización general se mantiene.

---

## Cómo podría utilizarse en una empresa

Un uso posible sería un proceso diario:

1. El sistema recibe los datos actualizados de ventas, promociones, precios e inventario.
2. Calcula las variables históricas de cada tienda y producto.
3. Aplica el modelo entrenado.
4. Asigna una probabilidad de riesgo.
5. Ordena los casos.
6. Genera una lista de alertas.
7. El planificador revisa los casos prioritarios.
8. Decide si debe adelantar un pedido, redistribuir stock o comprobar el inventario.

Ejemplo de salida:

| Fecha | Tienda | Producto | Probabilidad | Prioridad |
|---|---:|---:|---:|---|
| 2026-07-12 | 12 | 305 | 0,91 | Alta |
| 2026-07-12 | 8 | 114 | 0,84 | Alta |
| 2026-07-12 | 3 | 412 | 0,66 | Media |
| 2026-07-12 | 21 | 92 | 0,23 | Baja |

La acción final siempre debería depender también de:

- stock disponible;
- pedidos en tránsito;
- stock de seguridad;
- demanda promocional;
- caducidad;
- criticidad del producto;
- capacidad del almacén;
- criterio del planificador.

---

## Propuesta conceptual de puesta en producción

Este proyecto no implementa una aplicación productiva, pero plantea cómo podría desplegarse.

### Flujo diario

```text
WMS / ERP / Base de datos
          │
          ▼
Extracción diaria de datos
          │
          ▼
Validación y limpieza
          │
          ▼
Cálculo de variables
          │
          ▼
Aplicación del modelo
          │
          ▼
Listado de riesgos
          │
          ▼
Dashboard o herramienta operativa
```

### Monitorización necesaria

Deberían controlarse:

- calidad de los datos de entrada;
- valores ausentes;
- columnas inesperadas;
- cambios en la distribución de las variables;
- cambios en el porcentaje de alertas;
- precision y recall cuando exista el resultado real;
- tiempo de ejecución;
- versión del modelo;
- fecha del último entrenamiento.

### Reentrenamiento

El modelo podría reentrenarse:

- cada mes;
- cada trimestre;
- cuando se detecte una caída de rendimiento;
- cuando cambien de forma significativa la demanda, el surtido o la red logística.

### Versionado

Cada modelo debería guardar:

- fecha de entrenamiento;
- periodo utilizado;
- lista de variables;
- parámetros;
- métricas;
- versión del código;
- ubicación del archivo generado.

---

## Limitaciones

### 1. El target es sintético

No existen datos reales de rotura de stock.

### 2. No se dispone de inventario real

Faltan variables fundamentales como:

- stock disponible;
- stock reservado;
- stock en tránsito;
- pedidos pendientes;
- stock de seguridad real;
- frecuencia de reposición.

### 3. No se observan ventas perdidas

Cuando las ventas son cero, no puede saberse si no hubo demanda o si faltaba producto.

### 4. El plazo de reposición es simulado

En una empresa real debería obtenerse del proveedor, WMS, ERP o maestro logístico.

### 5. Las métricas económicas son escenarios

No representan beneficios reales hasta que se utilicen costes y resultados observados.

### 6. Capacidad predictiva moderada

Las métricas indican que el modelo aporta señal, pero no una separación perfecta entre riesgo y ausencia de riesgo.

### 7. No existe validación en producción

El proyecto evalúa los datos históricos preparados, pero no se ha probado dentro de una operación real.

---

## Mejoras futuras

Las principales mejoras serían:

1. Incorporar inventario real.
2. Añadir pedidos pendientes y mercancía en tránsito.
3. Utilizar plazos de reposición reales.
4. Registrar roturas de stock observadas.
5. Estimar ventas perdidas.
6. Añadir información de proveedores.
7. Incorporar festivos y eventos locales.
8. Añadir datos meteorológicos cuando sean relevantes.
9. Probar validación temporal con varias ventanas.
10. Calibrar las probabilidades del modelo.
11. Optimizar hiperparámetros con validación temporal.
12. Crear modelos específicos por familia de producto.
13. Comparar con modelos de series temporales.
14. Evaluar costes reales de falsos positivos y falsos negativos.
15. Crear un dashboard de seguimiento.
16. Automatizar la generación diaria de alertas.
17. Registrar las decisiones tomadas por los planificadores.
18. Medir el impacto real mediante una prueba piloto.

---

## Tecnologías utilizadas

- Python
- Jupyter Notebook
- Pandas
- NumPy
- Matplotlib
- Seaborn
- Scikit-learn
- XGBoost
- SHAP
- Joblib
- Git y GitHub

---

## Conclusión

El proyecto demuestra el desarrollo completo de una solución de Machine Learning aplicada a un problema logístico:

- se parte de datos sin preparar;
- se estudia su calidad y comportamiento;
- se construyen variables predictivas;
- se define un objetivo de clasificación;
- se entrenan y comparan modelos;
- se evalúan los resultados respetando el tiempo;
- se explican las predicciones;
- se analizan los errores;
- se traducen las métricas a decisiones operativas;
- se propone una posible evolución hacia producción.

El principal valor del sistema no consiste en afirmar con certeza cuándo se producirá una rotura de stock, sino en **ordenar los casos por riesgo para utilizar mejor el tiempo y los recursos del equipo logístico**.

La conclusión debe mantenerse dentro del alcance real de los datos: el proyecto representa una prueba académica completa y reproducible, pero necesitaría inventario y roturas observadas para validar su impacto en una operación real.
