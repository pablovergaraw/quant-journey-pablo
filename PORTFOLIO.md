# Portafolio Quant — Pablo Vergara

Repositorio de análisis cuantitativo aplicado al mercado accionario chileno. Cubre desde estadística descriptiva y análisis exploratorio hasta construcción de portafolios (Markowitz) y backtesting de estrategias.

---

## 01 — Análisis Exploratorio del Mercado Accionario Chileno

**Notebook:** `01_pandas_basico.ipynb`  
**Período:** Mayo 2024 – Mayo 2026  
**Universo:** 9 acciones del IPSA — COPEC, FALABELLA, CMPC, CENCOSUD, ENELCHILE, BSANTANDER, BCI, CAP, AGUAS-A  
**Benchmark:** ECH (iShares MSCI Chile ETF) como proxy del mercado chileno

### Metodología

- Descarga de precios históricos OHLCV vía Yahoo Finance (`yfinance`)
- Cálculo de retornos diarios (`pct_change`) y retornos acumulados normalizados (base 100)
- Comparación de retornos relativos al benchmark ECH
- Estadística descriptiva: media, desviación estándar, Sharpe simplificado, kurtosis
- Comparación distribución empírica (KDE) vs normal teórica (`scipy.stats.norm`)
- Test formal de normalidad: Jarque-Bera (`scipy.stats.jarque_bera`)
- Análisis de autocorrelación de retornos: scatter `r_t` vs `r_{t+1}`
- Test formal de autocorrelación: Ljung-Box lag=1 (`statsmodels.stats.diagnostic`)
- Análisis de agrupamiento de volatilidad: scatter `|r_t|` vs `|r_{t+1}|`
- Matriz de correlación entre activos (`seaborn.heatmap`)

### Resultados

**Retornos acumulados (2 años):**

| Acción | Retorno total | Sharpe |
|---|---|---|
| BCI.SN | +131% | 0.115 |
| FALABELLA.SN | +109% | 0.097 |
| BSANTANDER.SN | +68% | 0.090 |
| ENELCHILE.SN | +53% | 0.068 |
| ECH (benchmark) | +43% | 0.057 |
| AGUAS-A.SN | +33% | 0.053 |
| CENCOSUD.SN | +16% | 0.027 |
| CAP.SN | -4% | 0.004 |
| COPEC.SN | -15% | -0.014 |
| CMPC.SN | -44% | -0.060 |

**Hallazgos estadísticos:**

1. **Leptokurtosis generalizada:** todas las acciones presentan exceso de kurtosis positivo. El test Jarque-Bera rechaza normalidad en 9 de 10 activos con p << 0.05. COPEC.SN es la única excepción (p=0.107), consistente con su kurtosis de 0.39 — la más baja del universo. El VaR paramétrico subestima el riesgo real en la mayoría de los activos del IPSA.

2. **Eficiencia de mercado (EMH):** el test Ljung-Box (lag=1) no detecta autocorrelación estadísticamente significativa en 8 de 10 activos, consistente con la hipótesis de mercado eficiente en su forma débil. Las excepciones son BCI.SN (p=0.026, atribuible a menor liquidez) y CAP.SN (p=0.001, atribuible a *price discovery* lento respecto al precio del cobre en mercados internacionales). Estos activos son candidatos a estrategias de momentum de corto plazo.

3. **Agrupamiento de volatilidad:** correlación positiva entre `|r_t|` y `|r_{t+1}|` en todos los activos. Días volátiles tienden a agruparse, fenómeno explicado por la llegada de información en rachas y comportamiento de manada. Base empírica para modelos GARCH.

4. **Estructura sectorial en correlaciones:** mayor correlación dentro del mismo sector (banca: BSANTANDER–BCI; retail: FALABELLA–CENCOSUD). La diversificación intersectorial es más eficiente que la intrasectorial — principio central de Markowitz.

### Herramientas

`pandas` · `yfinance` · `matplotlib` · `seaborn` · `scipy` · `numpy` · `statsmodels`

---

*Próximo notebook: construcción de portafolio eficiente de Markowitz con las mismas 9 acciones. Notebook adicional: backtesting de estrategia de momentum en CAP.SN y BCI.SN a partir de la autocorrelación detectada.*
