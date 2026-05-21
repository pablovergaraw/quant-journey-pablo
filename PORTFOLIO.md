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

## 02 — Optimización de Portafolio: Modelo de Markowitz

**Notebook:** `02_markowitz.ipynb`  
**Período:** Mayo 2024 – Mayo 2026  
**Universo:** 9 acciones del IPSA — mismo universo del notebook 01  
**Benchmark:** ECH (iShares MSCI Chile ETF)

### Metodología

- Descarga de precios históricos vía `yfinance` y cálculo de retornos diarios
- Cálculo de vector de retornos medios y matriz de covarianza como insumos del modelo
- **Simulación de Monte Carlo:** 10.000 portafolios con pesos aleatorios normalizados; visualización de la nube riesgo-retorno con gradiente de color por Sharpe ratio
- **Optimización analítica — máximo Sharpe:** `scipy.optimize.minimize` minimizando el Sharpe negativo, con restricciones de suma unitaria y sin posiciones cortas
- **Optimización analítica — mínima varianza:** minimización de la volatilidad sin restricción de retorno; identifica el portafolio de menor riesgo posible dentro del universo
- Construcción de la **frontera eficiente analítica** minimizando volatilidad para 50 niveles de retorno objetivo; filtrado de la parte ineficiente por pendiente
- Trazado de la **Capital Market Line (CML):** recta desde la TPM (~5.5% anual) tangente a la frontera en el portafolio de máximo Sharpe
- Comparación con benchmark ECH (Sharpe diario)
- Anualización de métricas: retorno × 252, volatilidad × √252
- Cálculo de Sharpe real incorporando TPM como tasa libre de riesgo

### Resultados

**Portafolio óptimo (máximo Sharpe):**

| Acción | Peso |
|---|---|
| BCI | ~40.4% |
| Falabella | ~20.7% |
| BancoSantander | ~17.2% |
| Enelchile | ~12.1% |
| Aguas Andinas | ~9.7% |
| CAP, CMPC, COPEC, CENCOSUD | 0% |

**Métricas anualizadas — portafolio de máximo Sharpe:**

| Métrica | Valor |
|---|---|
| Sharpe ratio sin rf (anualizado) | ~2.10 |
| Sharpe ratio real (rf = 5.5%) | ~1.79 |
| Supera benchmark ECH en Sharpe | ~2.1x |

**Portafolio de mínima varianza:**

| Acción | Peso |
|---|---|
| Aguas Andinas | ~33.7% |
| BancoSantander | ~16.3% |
| Enelchile | ~15.7% |
| CAP | ~15.1% |
| CENCOSUD | ~8.0% |
| COPEC | ~7.8% |
| BCI | ~3.2% |
| CMPC, Falabella | ~0% |

El portafolio de mínima varianza concentra en AGUAS-A por ser el activo menos volátil del universo, diversificando el resto entre activos con baja covarianza. Sacrifica retorno esperado a cambio de minimizar la volatilidad total.

### Limitaciones

1. **Overfitting histórico:** el portafolio se calibra y evalúa sobre el mismo período. Los pesos óptimos pasados no garantizan rendimiento futuro.
2. **Supuesto de normalidad:** Markowitz asume distribución normal de retornos, subestimando eventos extremos — inconsistente con el notebook 01 (leptokurtosis generalizada).
3. **Parámetros estáticos:** la media y covarianza históricas pueden cambiar ante shocks macroeconómicos.

### Herramientas

`pandas` · `numpy` · `yfinance` · `scipy.optimize` · `matplotlib`

---

*Próximo notebook: backtesting con walk-forward validation — calibrar el modelo en un período y validar en el siguiente para estimar la robustez predictiva real. Notebook adicional: backtesting de estrategia de momentum en CAP.SN y BCI.SN a partir de la autocorrelación detectada en el notebook 01.*
