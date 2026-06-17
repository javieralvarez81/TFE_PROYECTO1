# Predicción del riesgo de rotura de stock en un entorno logístico

## 1. Contexto del proyecto

La gestión eficiente del stock es uno de los retos principales en entornos logísticos y de distribución. Una rotura de stock puede provocar pérdida de ventas, incumplimientos de servicio, deterioro de la experiencia del cliente y costes operativos adicionales derivados de reposiciones urgentes o replanificaciones.

Este proyecto desarrolla un sistema predictivo orientado a anticipar situaciones de riesgo de rotura de stock a partir de datos históricos de ventas, precios, promociones y patrones temporales. El objetivo no es sustituir los procesos de planificación de inventario, sino proporcionar una herramienta de apoyo que permita priorizar productos, tiendas o combinaciones con mayor probabilidad de presentar problemas de disponibilidad.

El trabajo se plantea como un proyecto *end-to-end* de Machine Learning aplicado a un caso logístico realista, desde el análisis exploratorio de datos hasta la generación de métricas técnicas y su traducción a indicadores de negocio.

---

## 2. Objetivo de negocio

El objetivo de negocio es identificar de forma anticipada situaciones con riesgo de rotura de stock para facilitar la toma de decisiones operativas.

Si el modelo funciona correctamente, podría aportar valor en los siguientes aspectos:

* Priorización de productos o tiendas con mayor riesgo.
* Reducción de roturas de stock no detectadas.
* Mejora de la planificación de reposición.
* Apoyo a equipos de operaciones, aprovisionamiento o planificación.
* Reducción de costes asociados a incidencias de disponibilidad.

El problema se formula como una tarea de clasificación binaria, donde la variable objetivo indica si existe o no riesgo de *stockout*.

---

## 3. Descripción del dataset

El dataset utilizado contiene información histórica de ventas en diferentes tiendas y productos. Las variables originales incluyen información como fecha, tienda, producto, ventas, precio y promociones.

A partir de estas variables se construyen nuevas características relacionadas con temporalidad, comportamiento histórico de la demanda y variabilidad de ventas.

Las principales variables utilizadas en el proyecto son:

* `sales`: unidades vendidas.
* `price`: precio del producto.
* `promo`: indicador de promoción.
* `weekday`: día de la semana.
* `month`: mes.
* `week`: semana del año.
* `sales_lag_7`, `sales_lag_14`, `sales_lag_30`: ventas retrasadas.
* `rolling_mean_7`, `rolling_mean_14`, `rolling_mean_30`: medias móviles de demanda.
* `rolling_std_30`: variabilidad reciente de la demanda.
* `trend_7_30`: diferencia entre demanda reciente y demanda de más largo plazo.
* `lead_time_days`: plazo estimado de reposición.
* `stockout_risk`: variable objetivo.

Debido al tamaño del dataset original, no se adjunta al proyecto. Se puede consultar y descargar  en:
https://www.kaggle.com/datasets/dhrubangtalukdar/store-item-demand-forecasting-dataset

---

## 4. Análisis exploratorio de datos

El análisis exploratorio se realizó en el notebook `01_eda.ipynb`.

Durante esta fase se analizaron:

* Ventas por fecha.
* Evolución temporal de ventas.
* Ventas medias por mes.
* Ventas medias por día de la semana.
* Distribución de ventas por producto.
* Distribución de precios.
* Impacto de promociones por tienda y producto.
* Variabilidad de la demanda.
* Posibles patrones estacionales.
* Relación entre ventas, precio y promoción.

Este análisis permitió justificar la creación de variables temporales, retardos de ventas, medias móviles y métricas de variabilidad de demanda. Estas variables son relevantes porque el riesgo de rotura de stock está relacionado con el comportamiento reciente de la demanda y con patrones temporales o comerciales como promociones y cambios de precio.

---

## 5. Ingeniería de variables

La ingeniería de variables se desarrolló en el notebook `02_feature_engineering.ipynb`.

Se generaron variables temporales a partir de la fecha:

* Año.
* Mes.
* Trimestre.
* Semana.
* Día de la semana.
* Día del mes.

También se generaron variables basadas en el histórico de ventas por combinación tienda-producto:

* Retardos de ventas a 7, 14 y 30 días.
* Medias móviles a 7, 14 y 30 días.
* Desviación estándar móvil.
* Tendencia entre demanda reciente y demanda de mayor plazo.

La variable objetivo `stockout_risk` se construyó comparando la demanda futura estimada a 7 días con una aproximación del stock disponible y stock de seguridad.

Para evitar *data leakage*, se excluyeron del modelado variables directamente relacionadas con la construcción del target, como:

* `future_demand_7d`
* `stock_estimated`
* `safety_stock`

Estas variables se utilizaron para definir el problema, pero no para entrenar los modelos predictivos.

---

## 6. Modelado predictivo

El modelado se realizó en el notebook `03_model_training.ipynb`.

Se compararon tres modelos:

### 6.1. Logistic Regression

Se utilizó como modelo *baseline* interpretable. Permite establecer una referencia inicial y facilita la interpretación general del comportamiento de las variables.

### 6.2. Random Forest

Se utilizó como modelo robusto basado en árboles, capaz de capturar relaciones no lineales y manejar interacciones entre variables.

### 6.3. XGBoost

Se utilizó como modelo avanzado de *boosting*, habitualmente competitivo en problemas tabulares y adecuado para capturar patrones complejos.

La partición entre entrenamiento y test se realizó de forma temporal, utilizando los datos más antiguos para entrenar y los datos posteriores para evaluar. Esta decisión es más realista en un problema logístico, ya que en producción el modelo se entrenaría con información pasada para predecir situaciones futuras.

---

## 7. Evaluación técnica

Los modelos se evaluaron mediante accuracy, precision, recall, F1-score y ROC-AUC. Además, se generaron matrices de confusión y curvas ROC para analizar el comportamiento de los modelos desde una perspectiva técnica y operativa.

La comparación final de modelos fue la siguiente:

| Modelo | Accuracy | Precision | Recall | F1-score | ROC-AUC |
|---|---:|---:|---:|---:|---:|
| Logistic Regression | 0.5725 | 0.6351 | 0.6067 | 0.6206 | 0.5890 |
| XGBoost | 0.5903 | 0.6730 | 0.5621 | 0.6126 | 0.6238 |
| Random Forest | 0.5899 | 0.6727 | 0.5615 | 0.6121 | 0.6227 |

### Interpretación operativa del modelo final

A partir de la matriz de confusión del modelo final, se obtienen los siguientes resultados operativos:

| Indicador | Valor |
|---|---:|
| Stockouts correctamente detectados | 312.044 |
| Stockouts no detectados | 202.258 |
| Alertas falsas generadas | 179.268 |

Estos resultados muestran que el modelo es capaz de detectar una parte relevante de los casos de riesgo, aunque todavía deja sin identificar un número considerable de stockouts. También genera falsas alertas, lo que implica que su uso en un entorno real debería ir acompañado de una estrategia de priorización y revisión operativa.

Desde el punto de vista de negocio, esto significa que el modelo puede ser útil como sistema de apoyo a la decisión, pero no como mecanismo automático de reposición sin supervisión.

Aunque XGBoost y Random Forest obtienen un ROC-AUC superior, la regresión logística presenta un recall más alto. En un contexto logístico, este punto es relevante porque el recall mide la capacidad del modelo para detectar casos reales de riesgo de rotura de stock.

La elección del modelo final depende del criterio de negocio. Si se prioriza la capacidad de detección de stockouts, la regresión logística ofrece un comportamiento interesante como modelo base. Si se prioriza una mejor separación general entre clases, XGBoost presenta mejor ROC-AUC.

En este proyecto se mantiene un enfoque prudente: los resultados muestran una capacidad predictiva moderada. El modelo no debe interpretarse como una solución definitiva de predicción de roturas de stock, sino como una herramienta de priorización que puede ayudar a identificar casos con mayor probabilidad de riesgo.

Los resultados finales se encuentran en:

```text
outputs/metrics/model_metrics.csv
```

Y las visualizaciones principales en:

```text
outputs/figures/
```

---

## 8. Explicabilidad del modelo

La explicabilidad del modelo se analizó mediante la importancia de variables. El objetivo fue identificar qué factores tenían mayor influencia en la predicción del riesgo de stockout y comprobar si el comportamiento del modelo era coherente desde el punto de vista logístico.

Las variables más relevantes fueron:

| Variable | Importancia |
|---|---:|
| rolling_mean_14 | 1.6194 |
| rolling_mean_30 | 0.9024 |
| rolling_mean_7 | 0.8530 |
| sales | 0.3797 |
| trend_7_30 | 0.2235 |

La presencia de variables como `rolling_mean_14`, `rolling_mean_30`, `rolling_mean_7` y `sales` entre las más importantes indica que el comportamiento histórico de la demanda tiene un peso relevante en la predicción del riesgo de stockout.

Esto es coherente con el problema de negocio: las roturas de stock suelen estar relacionadas con patrones recientes de consumo, incrementos de demanda, variabilidad temporal y diferencias entre demanda reciente y demanda histórica.

La variable `trend_7_30` también aporta información útil, ya que representa la diferencia entre la demanda reciente y una ventana temporal más amplia. Esta variable puede ayudar a detectar cambios de tendencia que podrían anticipar situaciones de riesgo.

---

## 9. Métricas de negocio

Además de las métricas técnicas, se realizó una simulación de impacto de negocio para analizar cómo podrían traducirse las predicciones del modelo a indicadores operativos.

Las métricas consideradas fueron:

- Stockouts detectados.
- Stockouts no detectados.
- Falsas alertas.
- Coste estimado de no detectar una rotura de stock.
- Coste estimado de revisar una alerta.
- Ahorro potencial estimado.
- Priorización de casos de mayor riesgo.

Al no disponer de costes reales de negocio, se definieron escenarios simulados. Estos escenarios no deben interpretarse como un impacto económico validado, sino como una forma de mostrar cómo el modelo podría evaluarse desde una perspectiva empresarial.

### Priorización de casos de riesgo

Una forma práctica de utilizar el modelo sería ordenar los casos por probabilidad predicha de stockout y revisar solo los casos con mayor riesgo.

| Top de casos revisados | Casos revisados | Stockouts capturados | Tasa de captura | Precisión de alertas |
|---:|---:|---:|---:|---:|
| 5 % | 44.625 | 30.914 | 0.0601 | 0.6929 |
| 10 % | 89.250 | 61.637 | 0.1199 | 0.6907 |
| 20 % | 178.500 | 121.354 | 0.2360 | 0.6799 |
| 30 % | 267.750 | 178.941 | 0.3479 | 0.6683 |

Estos resultados muestran que el modelo puede utilizarse como herramienta de priorización. Por ejemplo, revisando el 10 % de los casos con mayor riesgo se capturaría aproximadamente el 12 % de los stockouts reales, con una precisión de alerta cercana al 69 %.

Aunque la tasa de captura no es elevada, la precisión de las alertas prioritarias es razonable. Esto indica que el modelo puede aportar valor si se utiliza para ordenar riesgos, no necesariamente para automatizar decisiones finales.

## Interpretación de resultados

Los resultados obtenidos muestran que el problema de predicción de stockout es complejo con las variables disponibles. El rendimiento del modelo es moderado, especialmente en términos de ROC-AUC, lo que indica que la separación entre casos con riesgo y sin riesgo no es completamente clara.

Esta limitación puede deberse a varios factores:

- El dataset no contiene stock real observado.
- La variable objetivo se ha construido mediante una aproximación.
- No se dispone de incidencias reales de rotura de stock.
- No se incluyen variables como proveedor, calendario de reposición, lead time real, festivos, pedidos pendientes o restricciones logísticas.
- El comportamiento de la demanda puede estar influido por factores externos no presentes en los datos.

Aun así, el modelo permite construir una primera aproximación útil para priorizar casos de riesgo. Desde una perspectiva empresarial, su valor no estaría en sustituir la planificación de inventario, sino en servir como herramienta de apoyo para identificar combinaciones tienda-producto que merecen una revisión prioritaria.

---

## 10. Propuesta conceptual de producción

En un escenario real, el modelo podría desplegarse como un proceso batch diario.

El flujo propuesto sería:

1. Extracción diaria de datos de ventas, stock, promociones y calendario.
2. Generación automática de variables.
3. Aplicación del modelo entrenado.
4. Generación de una tabla de riesgos por tienda-producto.
5. Visualización en un dashboard operativo.
6. Priorización de alertas para equipos de aprovisionamiento o planificación.

El modelo podría reentrenarse mensualmente o cuando se detecte pérdida de rendimiento. Para ello se monitorizarían métricas como:

* Recall.
* Precision.
* Distribución de probabilidades.
* Tasa de alertas.
* Posibles cambios en la distribución de las variables de entrada.

También sería recomendable controlar el *data drift*, especialmente en variables como:

* Ventas.
* Promociones.
* Precios.
* Patrones temporales de demanda.

Una posible arquitectura sería:

```text
Fuentes de datos
      ↓
Proceso ETL diario
      ↓
Generación de variables
      ↓
Modelo predictivo entrenado
      ↓
Tabla de riesgos stockout
      ↓
Dashboard operativo
      ↓
Acciones de planificación y reposición
```
## 11. Conclusiones

El proyecto ha permitido desarrollar un pipeline completo de Machine Learning aplicado a la predicción del riesgo de rotura de stock en un entorno logístico.

Se ha trabajado desde el análisis exploratorio de datos hasta la generación de variables, entrenamiento de modelos, evaluación técnica, explicabilidad y traducción de resultados a métricas de negocio.

Los resultados muestran una capacidad predictiva moderada. El modelo no alcanza un rendimiento suficientemente alto como para ser utilizado de forma automática en decisiones críticas de reposición, pero sí puede aportar valor como herramienta de priorización.

La comparación entre modelos muestra que XGBoost y Random Forest ofrecen mejor ROC-AUC, mientras que la regresión logística obtiene un recall superior. Esta diferencia pone de manifiesto la importancia de elegir el modelo no solo por una métrica técnica aislada, sino por el objetivo operativo que se quiera priorizar.

Desde el punto de vista de negocio, el modelo puede ayudar a ordenar casos por probabilidad de riesgo y facilitar la revisión de aquellos con mayor prioridad. Sin embargo, para su aplicación real sería necesario incorporar datos reales de inventario, roturas históricas, lead times reales y costes operativos validados.

## 12. Limitaciones

Las principales limitaciones del proyecto son:

- El dataset no contiene información real de stock físico.
- No existen registros reales de rotura de stock observada.
- La variable objetivo `stockout_risk` se ha construido mediante una aproximación basada en demanda futura estimada, stock estimado y stock de seguridad.
- Las variables `stock_estimated` y `safety_stock` han sido simuladas y no se han usado como features para evitar data leakage.
- El rendimiento predictivo es moderado, especialmente en términos de ROC-AUC.
- El modelo genera un volumen relevante de falsas alertas.
- No se incluyen variables externas como festivos, campañas comerciales, proveedor, calendario de reposición, pedidos pendientes o restricciones operativas.
- Las métricas económicas se basan en escenarios simulados y no en costes reales de empresa.

Estas limitaciones no invalidan el proyecto, pero sí condicionan su interpretación. El sistema debe entenderse como una prueba de concepto académica y no como un modelo productivo validado para su uso real inmediato.