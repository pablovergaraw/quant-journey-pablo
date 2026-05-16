# Portafolio Quant — Pablo Vergara

Repositorio de análisis cuantitativo aplicado al mercado accionario chileno. Cubre desde estadística descriptiva y análisis exploratorio hasta construcción de portafolios (Markowitz) y backtesting de estrategias.

---

## 01 — Análisis Exploratorio del Mercado Accionario Chileno

**Notebook:** `01_pandas_basico.ipynb`  
**Período:** Mayo 2024 – Mayo 2026  
**Universo:** 9 acciones del IPSA — COPEC, FALABELLA, CMPC, CENCOSUD, ENELCHILE, BSANTANDER, BCI, CAP, AGUAS-A

### Metodología

- Descarga de precios históricos OHLCV vía Yahoo Finance (`yfinance`)
- Cálculo de retornos diarios (`pct_change`) y retornos acumulados normalizados (base 100)
- Estadística descriptiva: media, desviación estándar, Sharpe simplificado, kurtosis
- Comparación distribución empírica (KDE) vs normal teórica (`scipy.stats.norm`)
- Análisis de autocorrelación de retornos: scatter `r_t` vs `r_{t+1}`
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
| AGUAS-A.SN | +33% | 0.053 |
| CENCOSUD.SN | +16% | 0.027 |
| CAP.SN | -4% | 0.004 |
| COPEC.SN | -15% | -0.014 |
| CMPC.SN | -44% | -0.060 |

**Hallazgos estadísticos:**

1. **Leptokurtosis generalizada:** todas las acciones presentan exceso de kurtosis positivo, con AGUAS-A en 5.12. Las distribuciones de retorno son más picudas y tienen colas más gruesas que la normal teórica — el VaR paramétrico subestima el riesgo real.

2. **Eficiencia de mercado (EMH):** sin autocorrelación en retornos diarios. El scatter `r_t` vs `r_{t+1}` no muestra estructura en ninguna de las 9 acciones, consistente con la hipótesis de mercado eficiente en su forma débil.

3. **Agrupamiento de volatilidad:** correlación positiva entre `|r_t|` y `|r_{t+1}|`. Días volátiles tienden a agruparse, fenómeno explicado por la llegada de información en rachas y comportamiento de manada. Base empírica para modelos GARCH.

4. **Estructura sectorial en correlaciones:** mayor correlación dentro del mismo sector (banca: BSANTANDER–BCI; retail: FALABELLA–CENCOSUD). La diversificación intersectorial es más eficiente que la intrasectorial.

### Herramientas

`pandas` · `yfinance` · `matplotlib` · `seaborn` · `scipy` · `numpy`

---

*Próximo notebook: construcción de portafolio eficiente de Markowitz con las mismas 9 acciones.*
