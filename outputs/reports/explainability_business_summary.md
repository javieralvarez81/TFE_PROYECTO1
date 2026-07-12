# Resumen ejecutivo — explicabilidad y negocio

## Rendimiento técnico
- ROC-AUC: 0.9842
- PR-AUC: 0.9973
- Precision con umbral 0,50: 0.9737
- Recall con umbral 0,50: 0.9621
- F1 con umbral 0,50: 0.9678

## Operación con umbral 0,50
- Alertas: 167667
- Riesgos detectados: 163251
- Riesgos no detectados: 6430
- Falsas alertas: 4416

## Variables más relevantes por permutación
- lead_time_days
- week
- rolling_std_30
- promo_days_last_7
- trend_7_30

## Selección de umbral
- Mejor umbral técnico por F1: 0.475
- Mejor umbral simulado en escenario base: 0.150
- Beneficio neto simulado en escenario base: 8,288,140.00
- ROI simulado en escenario base: 4.4841

## Limitación
- El target y los costes son simulados. El proyecto demuestra metodología y valor potencial, no impacto económico real.
- Antes de producción se necesitarían datos reales de inventario, pedidos, reposiciones, ventas perdidas y lead time.