# Diccionario de Datos - APEAJAL Avocado Project

**Fecha:** 2026-01-22  
**Preparado por:** Data Engineer Agent  
**Ubicación:** `data/clean/`

---

## 📊 Datasets Disponibles

### 1. tipo_cambio_diario_clean.csv

**Descripción:** Tipo de cambio USD/MXN diario con interpolación para días no hábiles.

**Fuente:** Banco de México (Banxico) - Serie SF43718

**Periodo:** 2022-01-03 a 2026-01-21 (1,480 días)

**Columnas:**

| Columna | Tipo | Descripción | Ejemplo |
|---------|------|-------------|---------|
| `date` | datetime64 | Fecha en formato YYYY-MM-DD | 2022-01-03 |
| `tipo_cambio` | float64 | Tipo de cambio FIX (MXN por USD) | 20.5890 |
| `source` | string | Fuente de datos | banxico |

**Estadísticas:**
- Rango: $16.34 - $21.38 MXN/USD
- Promedio: ~$19.50 MXN/USD
- Valores nulos: 0 (interpolados)

**Notas:**
- Días hábiles originales: 1,019
- Fines de semana y festivos interpolados linealmente
- Útil para convertir precios USD a MXN

**Uso recomendado:**
```python
import pandas as pd
df = pd.read_csv('data/clean/tipo_cambio_diario_clean.csv', parse_dates=['date'])
# Merge con otros datasets por fecha
```

---

### 2. produccion_anual_clean.csv

**Descripción:** Producción anual de aguacate por estado (Jalisco y Michoacán).

**Fuente:** SIAP (Servicio de Información Agroalimentaria y Pesquera)

**Periodo:** 2022-2024 (3 años)

**Columnas:**

| Columna | Tipo | Descripción | Ejemplo |
|---------|------|-------------|---------|
| `year` | int64 | Año de producción | 2024 |
| `month` | NA | Mes (NA para datos anuales) | NA |
| `state` | string | Estado productor | Jalisco |
| `production_tons` | float64 | Producción en toneladas | 339015.85 |
| `area_ha` | float64 | Superficie cosechada en hectáreas | 28409.64 |
| `yield_ton_ha` | float64 | Rendimiento (ton/ha) | 11.93 |
| `source` | string | Fuente de datos | SIAP |

**Estadísticas 2024:**
- **Jalisco:** 339,016 ton (14.5% nacional), 28,410 ha, 11.93 ton/ha
- **Michoacán:** 2,004,926 ton (85.5% nacional), 179,634 ha, 11.16 ton/ha
- **Total nacional:** 2,343,942 ton

**Notas:**
- ⚠️ Michoacán 2023 tiene datos incompletos (production_tons = NaN)
- Jalisco tiene mejor rendimiento por hectárea que Michoacán
- Datos agregados a nivel anual, sin desglose mensual

**Uso recomendado:**
```python
import pandas as pd
df = pd.read_csv('data/clean/produccion_anual_clean.csv')
# Filtrar por estado
jalisco = df[df['state'] == 'Jalisco']
```

---

### 3. precios_mensuales_clean.csv

**Descripción:** Precios promedio mensuales de aguacate Hass en centrales de abasto de México.

**Fuente:** SNIIM (Sistema Nacional de Información e Integración de Mercados)

**Periodo:** 2022-01-15 a 2026-01-15 (48 meses)

**Columnas:**

| Columna | Tipo | Descripción | Ejemplo |
|---------|------|-------------|---------|
| `date` | datetime64 | Fecha (día 15 de cada mes) | 2022-01-15 |
| `market` | string | Nombre de la central de abasto | Mercado de Abasto de Guadalajara |
| `product` | string | Tipo de producto | aguacate_hass |
| `price_avg` | float64 | Precio promedio (MXN/kg) | 60.39 |
| `source` | string | Fuente de datos | SNIIM |

**Estadísticas:**
- Registros: 1,460 (45 mercados × ~32 meses)
- Precio promedio: $60.39 MXN/kg
- Rango: $20.67 - $132.48 MXN/kg
- Alta variabilidad sugiere estacionalidad fuerte

**Mercados principales:**
- Mercado de Abasto de Guadalajara (Jalisco)
- Central de Abasto de Iztapalapa (CDMX)
- Mercado de Abasto de Morelia (Michoacán)

**Notas:**
- Algunos meses de 2022 tienen cobertura limitada (1 mercado)
- Precios en pesos mexicanos por kilogramo
- Datos del día 15 de cada mes (snapshot mensual)

**Uso recomendado:**
```python
import pandas as pd
df = pd.read_csv('data/clean/precios_mensuales_clean.csv', parse_dates=['date'])
# Filtrar mercados principales
principales = df[df['market'].isin([
    'Mercado de Abasto de Guadalajara',
    'Central de Abasto de Iztapalapa',
    'Mercado de Abasto de Morelia'
])]
# Calcular precio promedio nacional
precio_nacional = df.groupby('date')['price_avg'].mean()
```

---

### 4. mhaia_cosecha_semanal_clean.csv

**Descripción:** Datos semanales de cosecha de aguacate en México (principalmente Michoacán y Jalisco).

**Fuente:** MHAIA (Mexican Hass Avocado Importers Association) - Reportes semanales

**Periodo:** 2023-07-09 a 2026-06-28 (147 semanas)

**Columnas:**

| Columna | Tipo | Descripción | Ejemplo |
|---------|------|-------------|---------|
| `date` | datetime64 | Fecha de la semana | 2024-01-07 |
| `harvest_tons_projected` | float64 | Toneladas proyectadas de cosecha | 30,907 |
| `harvest_tons_actual` | float64 | Toneladas reales de cosecha (NaN si no disponible) | 24,954 |
| `source_report` | string | Fecha del reporte fuente | 2024-09-08 |
| `source` | string | Fuente de datos | mhaia |

**Estadísticas:**
- Registros: 147 semanas
- Con datos actuales: 122 (83%)
- Solo proyección: 25 (17%)
- Promedio semanal: 23,843 tons
- Rango: 3,968 - 43,413 tons
- Total anual estimado: ~1.24 millones tons

**Notas:**
- Datos consolidados de 130 reportes PDF (1,295 tablas CSV)
- Incluye proyecciones futuras (hasta junio 2026)
- Valores > 100,000 tons fueron filtrados (eran libras, no toneladas)
- Datos principalmente de Michoacán (mayor productor)

**Uso recomendado:**
```python
import pandas as pd
df = pd.read_csv('data/clean/mhaia_cosecha_semanal_clean.csv', parse_dates=['date'])
# Filtrar solo datos actuales
actual = df[df['harvest_tons_actual'].notna()]
# Calcular promedio mensual
df['month'] = df['date'].dt.to_period('M')
monthly = df.groupby('month')['harvest_tons_actual'].sum()
```

---

### 5. mhaia_embarques_semanal_clean.csv

**Descripción:** Datos semanales de embarques (exportaciones) de aguacate mexicano, principalmente a EE.UU.

**Fuente:** MHAIA (Mexican Hass Avocado Importers Association) - Reportes semanales

**Periodo:** 2023-07-09 a 2026-06-28 (147 semanas)

**Columnas:**

| Columna | Tipo | Descripción | Ejemplo |
|---------|------|-------------|---------|
| `date` | datetime64 | Fecha de la semana | 2024-01-07 |
| `shipment_tons_projected` | float64 | Toneladas proyectadas de embarque | 25,000 |
| `shipment_tons_actual` | float64 | Toneladas reales embarcadas (NaN si no disponible) | 22,500 |
| `source_report` | string | Fecha del reporte fuente | 2024-09-08 |
| `source` | string | Fuente de datos | mhaia |

**Estadísticas:**
- Registros: 147 semanas
- Con datos actuales: 122 (83%)
- Solo proyección: 25 (17%)
- Promedio semanal: 20,581 tons
- Rango: 4,834 - 32,781 tons

**Notas:**
- Representa la **DEMANDA** de exportación (principalmente EE.UU.)
- Ratio embarques/cosecha: ~86% (el resto es consumo interno)
- Útil para modelar demanda y correlacionar con precios

**Uso recomendado:**
```python
import pandas as pd
df = pd.read_csv('data/clean/mhaia_embarques_semanal_clean.csv', parse_dates=['date'])
# Filtrar solo datos actuales
actual = df[df['shipment_tons_actual'].notna()]
```

---

## 🔄 Relaciones Entre Datasets

### Análisis de Precios vs Producción
```python
# Merge producción anual con precios mensuales
precios['year'] = precios['date'].dt.year
merged = precios.merge(produccion, on='year', how='left')
```

### Análisis de Precios en USD
```python
# Convertir precios MXN a USD
precios_usd = precios.merge(tipo_cambio, left_on='date', right_on='date')
precios_usd['price_usd'] = precios_usd['price_avg'] / precios_usd['tipo_cambio']
```

---

## 📈 Casos de Uso

### 1. Análisis de Estacionalidad de Precios
```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv('data/clean/precios_mensuales_clean.csv', parse_dates=['date'])
df['month'] = df['date'].dt.month
estacionalidad = df.groupby('month')['price_avg'].mean()
estacionalidad.plot(kind='bar', title='Precio Promedio por Mes')
```

### 2. Comparación Jalisco vs Michoacán
```python
prod = pd.read_csv('data/clean/produccion_anual_clean.csv')
prod.pivot(index='year', columns='state', values='production_tons').plot(kind='bar')
```

### 3. Impacto del Tipo de Cambio en Precios
```python
precios = pd.read_csv('data/clean/precios_mensuales_clean.csv', parse_dates=['date'])
tc = pd.read_csv('data/clean/tipo_cambio_diario_clean.csv', parse_dates=['date'])
merged = precios.merge(tc, on='date')
merged.plot(x='tipo_cambio', y='price_avg', kind='scatter')
```

---

## ⚠️ Limitaciones y Consideraciones

### Calidad de Datos
1. **Michoacán 2023:** Datos de producción incompletos (NaN)
2. **SNIIM 2022:** Algunos meses con cobertura limitada
3. **Interpolación:** Tipo de cambio interpolado para días no hábiles

### Granularidad Temporal
- **Producción:** Anual (sin desglose mensual)
- **Precios:** Mensual (día 15)
- **Tipo de cambio:** Diario

### Cobertura Geográfica
- **Producción:** Solo Jalisco y Michoacán (~99% producción nacional)
- **Precios:** 45 mercados en todo México

---

## 🚀 Próximos Pasos

### Datos Pendientes de Consolidación
- **MHAIA:** 1,295 tablas CSV con datos semanales de cosecha (2023-2026)
  - Requiere análisis de formatos variables
  - Contiene distribución regional, pronósticos, embarques

---

## 6. usitc_importaciones_clean.csv

**Descripción:** Importaciones mensuales de aguacate mexicano a Estados Unidos (valor y cantidad).

**Periodo:** 2022-01-01 a 2025-10-01 (46 meses)

**Fuente:** USITC DataWeb (U.S. International Trade Commission)

**Registros:** 46

**Columnas:**
- `date` (datetime): Fecha (primer día del mes)
- `value_usd` (float): Valor total de importaciones en USD
- `quantity_kg` (float): Cantidad total importada en kilogramos
- `price_per_kg_usd` (float): Precio promedio por kilogramo (calculado)
- `source` (string): Fuente de datos = "usitc"

**Estadísticas:**
- Valor promedio: $257M USD/mes
- Cantidad promedio: 87M kg/mes (87,000 tons/mes)
- Precio promedio: $3.10 USD/kg
- Precio mínimo: $1.77 USD/kg
- Precio máximo: $5.59 USD/kg

**Notas:**
- Incluye 3 categorías HTS: 804400020 (Hass orgánico), 804400040 (Hass no orgánico), 804400090 (otros)
- Datos agregados por mes (suma de todas las categorías)
- Representa demanda real del mercado estadounidense
- Útil para correlacionar con producción mexicana y precios SNIIM

---

### Mejoras Sugeridas
1. ~~Agregar datos de exportaciones (USDA)~~ ✅ Completado con USITC
2. Incluir datos climáticos (CONAGUA)
3. ~~Consolidar datos MHAIA en series temporales semanales~~ ✅ Completado

---

## 📝 Metadatos

**Datasets limpios:** 6  
**Registros totales:** 3,139  
**Periodo cubierto:** 2022-2026  
**Tamaño total:** ~200 KB  

**Proceso de limpieza:**
- Conversión de formatos de fecha
- Interpolación de valores faltantes (tipo de cambio)
- Estandarización de nombres de columnas
- Validación de rangos y tipos de datos
- Agregación de categorías HTS para USITC

**Scripts utilizados:**
- Limpieza manual en Python (inline)
- `scripts/consolidar_mhaia.py` para datos MHAIA
- Ubicación: Comandos ejecutados en sesión interactiva

---

**Última actualización:** 2026-01-28 12:12  
**Preparado por:** Data Engineer Agent  
**Estado:** ✅ Listo para análisis y modelado
