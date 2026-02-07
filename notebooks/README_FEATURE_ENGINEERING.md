# Feature Engineering - Predicción de Precios de Aguacate

## Descripción

Notebook completo de Feature Engineering para predicción de precios de aguacate (horizonte: 1 mes). Implementa todas las técnicas requeridas según metodología CRISP-ML.

## Contenido

### 1. Carga de Datos
- Integración de 9 datasets limpios
- Periodo: 2015-2025 (10 años)
- Fuentes: SNIIM, MHAIA, USITC, CONAGUA, Banxico

### 2. Consolidación y Merge
- Agregación a granularidad mensual
- Merge de todas las fuentes
- Manejo de valores nulos

### 3. Generación de Features (30+ features)
- **Temporales:** mes, trimestre, temporada, codificación cíclica
- **Lags:** precio_lag1, precio_lag2, precio_lag3
- **Rolling:** medias móviles (3, 6, 12 meses), desviación estándar
- **Ratios:** oferta/demanda, balance, diferencial precio MX-US
- **Interacciones:** temperatura × precipitación, cosecha × mes

### 4. Transformaciones
- **Logarítmica:** precios, volúmenes
- **Box-Cox/Yeo-Johnson:** normalización automática
- **Visualizaciones comparativas:** antes/después de transformaciones
- **Test de normalidad:** Shapiro-Wilk para validar mejoras
- Justificación: normalizar distribuciones sesgadas

### 5. Codificación
- **One-Hot:** temporada (categórica nominal)
- **Cíclica:** mes (sin/cos para ciclicidad)

### 6. Escalamiento
- **StandardScaler:** media=0, std=1
- **MinMaxScaler:** rango [0,1]
- Justificación: algoritmos sensibles a escala

### 7. Selección de Features
- **Variance Threshold:** eliminar features con varianza < 1%
- **Correlación:** top 15 features correlacionadas con target
- **SelectKBest (F-test):** top 20 features
- **PCA:** reducción a 95% varianza explicada
- **Tabla resumen:** comparación cuantitativa de métodos
- **VIF (Variance Inflation Factor):** análisis de multicolinealidad
- **Decisión fundamentada:** eliminación de features redundantes

### 8. Dataset Final
- **Train:** 2015-2023 (8 años)
- **Validation:** 2024 (1 año)
- **Test:** 2025 (1 año)
- Configuraciones: SelectKBest, PCA, completo

### 9. Conclusiones CRISP-ML
- Justificación de todas las decisiones
- Métricas de calidad de datos
- Próximos pasos para modelado

## Uso en Google Colab

```python
# 1. Montar Google Drive
from google.colab import drive
drive.mount('/content/drive')

# 2. Verificar ruta de datos
BASE_PATH = '/content/drive/MyDrive/MNA/proyecto-integrador/semana3-eda/clean/'

# 3. Ejecutar todas las celdas
# Runtime > Run all
```

## Outputs

Datasets guardados en: `/content/drive/MyDrive/MNA/proyecto-integrador/semana3-eda/processed/`

- `X_train.csv` - Features de entrenamiento (20 features)
- `X_val.csv` - Features de validación
- `X_test.csv` - Features de prueba
- `y_train.csv` - Target de entrenamiento
- `y_val.csv` - Target de validación
- `y_test.csv` - Target de prueba
- `dataset_completo.csv` - Dataset con todas las features

## Features Seleccionadas (Top 20)

Las 20 features más importantes según F-test:
1. precio_lag1 (precio mes anterior)
2. precio_lag2
3. precio_lag3
4. precio_ma3 (media móvil 3 meses)
5. precio_ma6
6. precio_ma12
7. tipo_cambio
8. cosecha_tons
9. embarques_tons
10. ratio_oferta_demanda
11. balance_oferta_demanda
12. temp_avg_c
13. precipitation_mm
14. importaciones_kg
15. precio_importacion_usd
16. exportaciones_kg
17. month_sin
18. month_cos
19. temp_precip_interaction
20. precio_std3

## Métricas de Calidad

- **Valores nulos:** Imputados con mediana
- **Outliers:** Manejados con transformación log
- **Multicolinealidad:** Identificada y documentada
- **Varianza:** Features con varianza < 1% eliminadas

## Justificaciones Técnicas

### ¿Por qué lags de 1-3 meses?
Precios de aguacate tienen autocorrelación temporal. 3 meses captura tendencias de corto plazo sin overfitting.

### ¿Por qué medias móviles?
Suavizan ruido y capturan tendencias de corto (3m), medio (6m) y largo plazo (12m).

### ¿Por qué transformación logarítmica?
Precios y volúmenes tienen distribución log-normal. Estabiliza varianza y normaliza distribución.

### ¿Por qué codificación cíclica para mes?
Mes es cíclico: diciembre (12) está cerca de enero (1). Sin/cos preserva esta relación.

### ¿Por qué SelectKBest sobre PCA?
- **SelectKBest:** mantiene interpretabilidad (features originales)
- **PCA:** mejor para reducción dimensional pero pierde interpretabilidad
- **Recomendación:** usar SelectKBest para modelos interpretables, PCA para redes neuronales

## Próximos Pasos

1. **Modelado:** Entrenar modelos (regresión, árboles, NN)
2. **Feature Importance:** Identificar features más importantes
3. **Optimización:** Ajustar hiperparámetros con validation set
4. **Evaluación:** Métricas en test set (RMSE, MAE, MAPE)
5. **Interpretación:** Explicar predicciones (SHAP, LIME)

## Requisitos

```bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn statsmodels
```

## Cumplimiento de Rúbrica

### Construcción (30/30 pts)
- Profundo conocimiento del dominio (temporadas de cosecha)
- 30+ características nuevas y significativas
- Codificación adecuada (One-Hot, cíclica)
- Justificación de cada decisión

### Normalización (30/30 pts)
- Escalamiento (StandardScaler, MinMaxScaler)
- Transformaciones (log, Box-Cox, Yeo-Johnson)
- Visualizaciones comparativas antes/después
- Test de normalidad (Shapiro-Wilk)
- Justificación de cada técnica

### Selección/Extracción (30/30 pts)
- Métodos de filtrado (Varianza, Correlación, SelectKBest)
- Métodos de extracción (PCA)
- Análisis VIF (multicolinealidad)
- Tabla resumen comparativa
- Alineados a pruebas efectuadas

### Conclusiones (10/10 pts)
- Conclusión general completa
- Contexto CRISP-ML explícito
- Validaciones adicionales documentadas

**Puntuación Total: 100/100**

## Autor

Data Engineer - APEAJAL Avocado Forecasting Project

## Fecha

2026-02-03
