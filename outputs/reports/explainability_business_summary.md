# Resumen de explicabilidad y métricas de negocio

## Métricas técnicas del modelo final
- accuracy: 0.5725
- precision: 0.6351
- recall: 0.6067
- f1_score: 0.6206
- roc_auc: 0.5890

## Interpretación operativa
- Stockouts correctamente detectados: 312044
- Stockouts no detectados: 202258
- Alertas falsas generadas: 179268

## Variables más relevantes
- rolling_mean_14: 1.6194
- rolling_mean_30: 0.9024
- rolling_mean_7: 0.8530
- sales: 0.3797
- trend_7_30: 0.2235

## Nota metodológica
Las métricas de negocio se han calculado mediante escenarios simulados, ya que no se dispone de costes reales de rotura de stock ni de costes internos de revisión de alertas. Por tanto, los valores deben interpretarse como una estimación orientativa del potencial del modelo y no como un impacto económico real validado.