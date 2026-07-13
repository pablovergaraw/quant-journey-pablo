# Proyecto 01 — Análisis Exploratorio del Mercado Accionario Chileno

## Objetivo

Construir un pipeline de análisis de datos sobre acciones del IPSA (Índice de Precio Selectivo de Acciones), el principal índice bursátil de Chile, con el fin de caracterizar el comportamiento histórico de los activos y establecer la base para modelos de optimización de portafolios.

## Activos analizados

| Ticker | Empresa | Sector |
|---|---|---|
| COPEC.SN | Copec S.A. | Energía |
| FALABELLA.SN | S.A.C.I. Falabella | Retail |
| CMPC.SN | Empresas CMPC S.A. | Forestal / Papel |
| CENCOSUD.SN | Cencosud S.A. | Retail |
| ENELCHILE.SN | Enel Chile S.A. | Utilities |
| BSANTANDER.SN | Banco Santander Chile | Banca |
| BCI.SN | Banco de Crédito e Inversiones | Banca |
| CAP.SN | CAP S.A. | Minería |
| AGUAS-A.SN | Aguas Andinas S.A. | Utilities |

**Benchmark:** ECH (iShares MSCI Chile ETF) · **Período:** Julio 2024 – Julio 2026 · **Frecuencia:** Diaria · **Fuente:** Yahoo Finance API

## Metodología

1. Extracción de precios históricos OHLCV vía `yfinance`
2. Aislamiento de precios de cierre ajustados
3. Normalización de series para comparación de rendimiento relativo (base 100)
4. Cálculo de retornos acumulados, retornos diarios y Sharpe ratio simplificado
5. Comparación de retornos acumulados contra el benchmark ECH
6. Análisis de distribución: KDE empírica vs normal teórica, kurtosis, test Jarque-Bera
7. Análisis de autocorrelación: retorno t vs t+1, test Ljung-Box
8. Análisis de agrupamiento de volatilidad: |retorno| t vs t+1
9. Matriz de correlación entre activos

## Resultados

### Retorno acumulado y Sharpe anualizado (orden por Sharpe)

| Activo | Retorno Acumulado | Sharpe anual |
|---|---|---|
| BCI.SN | +153.7% | 1.94 |
| BSANTANDER.SN | +92.6% | 1.72 |
| FALABELLA.SN | +109.2% | 1.50 |
| ENELCHILE.SN | +67.0% | 1.26 |
| ECH (benchmark) | +55.2% | 1.07 |
| AGUAS-A.SN | +41.3% | 0.99 |
| CENCOSUD.SN | +22.0% | 0.51 |
| CAP.SN | +9.7% | 0.31 |
| COPEC.SN | -10.2% | -0.11 |
| CMPC.SN | -33.9% | -0.62 |

Todos los activos superaron al benchmark ECH salvo COPEC y CMPC, los dos con desempeño negativo en el período.

### Distribución de retornos y normalidad

Los 10 activos (9 acciones + ECH) presentan exceso de kurtosis positivo (leptokurtosis), desde 0.54 (CMPC) hasta 4.79 (AGUAS-A). El test Jarque-Bera **rechaza normalidad en los 10 de 10 activos** (p < 0.05 en todos los casos), con AGUAS-A como el caso más extremo (p≈6.6×10⁻¹⁰¹). El VaR paramétrico, que asume normalidad, subestima sistemáticamente el riesgo de cola en este universo.

### Eficiencia de mercado (EMH)

El test Ljung-Box (lag=1) no detecta autocorrelación estadísticamente significativa en 8 de 10 activos, consistente con la Hipótesis de Mercado Eficiente en su forma débil. Las excepciones son **BCI.SN** (p=0.0079, atribuible a menor liquidez) y **CAP.SN** (p=0.0094, atribuible a *price discovery* lento respecto al precio del cobre en mercados internacionales) — candidatos a estrategias de momentum de corto plazo.

### Agrupamiento de volatilidad

Correlación positiva entre `|r_t|` y `|r_{t+1}|` en todos los activos: los días volátiles tienden a agruparse, fenómeno consistente con llegada de información en rachas y comportamiento de manada. Es la base empírica de los modelos GARCH.

### Estructura sectorial en correlaciones

Mayor correlación dentro del mismo sector (banca: BSANTANDER–BCI; retail: FALABELLA–CENCOSUD). La diversificación intersectorial es más eficiente que la intrasectorial — principio central de Markowitz, desarrollado en el Proyecto 02.

## Limitaciones

- Ventana de dos años: puede no capturar regímenes de mercado de largo plazo (recesiones, ciclos de tasas completos).
- Resultados marginales (p-value cercano a 0.05) son sensibles al tamaño de muestra exacto; no deben sobre-interpretarse como hallazgos robustos sin validación adicional.

## Herramientas

`pandas` · `yfinance` · `matplotlib` · `seaborn` · `scipy` · `numpy` · `statsmodels`
