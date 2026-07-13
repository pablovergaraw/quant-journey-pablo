# Portafolio Quant — Pablo Vergara

Repositorio de análisis cuantitativo aplicado al mercado accionario chileno. Cubre desde estadística descriptiva y análisis exploratorio hasta construcción de portafolios (Markowitz) y backtesting de estrategias.

---

## 01 — Análisis Exploratorio del Mercado Accionario Chileno

**Notebook:** `01_pandas_basico.ipynb`  
**Período:** Julio 2024 – Julio 2026  
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

| Acción | Retorno total | Sharpe anual |
|---|---|---|
| BCI.SN | +154% | 1.94 |
| BSANTANDER.SN | +93% | 1.72 |
| FALABELLA.SN | +109% | 1.50 |
| ENELCHILE.SN | +67% | 1.26 |
| ECH (benchmark) | +55% | 1.07 |
| AGUAS-A.SN | +41% | 0.99 |
| CENCOSUD.SN | +22% | 0.51 |
| CAP.SN | +10% | 0.31 |
| COPEC.SN | -10% | -0.11 |
| CMPC.SN | -34% | -0.62 |

**Hallazgos estadísticos:**

1. **Leptokurtosis generalizada:** todas las acciones presentan exceso de kurtosis positivo (entre 0.54 y 4.79). El test Jarque-Bera rechaza normalidad en **10 de 10 activos** con p << 0.05, incluyendo COPEC.SN — que en corridas anteriores del proyecto era la única excepción. El VaR paramétrico subestima el riesgo real en todo el universo del IPSA analizado.

2. **Eficiencia de mercado (EMH):** el test Ljung-Box (lag=1) no detecta autocorrelación estadísticamente significativa en 8 de 10 activos, consistente con la hipótesis de mercado eficiente en su forma débil. Las excepciones son BCI.SN (p=0.008, atribuible a menor liquidez) y CAP.SN (p=0.009, atribuible a *price discovery* lento respecto al precio del cobre en mercados internacionales). Estos activos son candidatos a estrategias de momentum de corto plazo.

3. **Agrupamiento de volatilidad:** correlación positiva entre `|r_t|` y `|r_{t+1}|` en todos los activos. Días volátiles tienden a agruparse, fenómeno explicado por la llegada de información en rachas y comportamiento de manada. Base empírica para modelos GARCH.

4. **Estructura sectorial en correlaciones:** mayor correlación dentro del mismo sector (banca: BSANTANDER–BCI; retail: FALABELLA–CENCOSUD). La diversificación intersectorial es más eficiente que la intrasectorial — principio central de Markowitz.

### Herramientas

`pandas` · `yfinance` · `matplotlib` · `seaborn` · `scipy` · `numpy` · `statsmodels`

---

## 02 — Optimización de Portafolio: Modelo de Markowitz

**Notebook:** `02_markowitz.ipynb`  
**Período:** Julio 2024 – Julio 2026  
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
| BCI | ~37.5% |
| Falabella | ~25.6% |
| BancoSantander | ~15.9% |
| Aguas Andinas | ~12.3% |
| Enelchile | ~8.8% |
| CAP, CMPC, COPEC, CENCOSUD | 0% |

**Métricas anualizadas — portafolio de máximo Sharpe:**

| Métrica | Valor |
|---|---|
| Sharpe ratio sin rf (anualizado) | ~2.09 |
| Sharpe ratio real (rf = 5.5%) | ~1.78 |
| Supera benchmark ECH en Sharpe | ~2.19x |

**Portafolio de mínima varianza:**

| Acción | Peso |
|---|---|
| Aguas Andinas | ~33.6% |
| BancoSantander | ~16.3% |
| Enelchile | ~15.7% |
| CAP | ~15.1% |
| CENCOSUD | ~8.1% |
| COPEC | ~7.9% |
| BCI | ~3.2% |
| CMPC, Falabella | ~0% |

El portafolio de mínima varianza concentra en AGUAS-A por ser el activo menos volátil del universo, diversificando el resto entre activos con baja covarianza. Sacrifica retorno esperado a cambio de minimizar la volatilidad total.

### Limitaciones

1. **Overfitting histórico:** el portafolio se calibra y evalúa sobre el mismo período. Los pesos óptimos pasados no garantizan rendimiento futuro.
2. **Supuesto de normalidad:** Markowitz asume distribución normal de retornos, subestimando eventos extremos — inconsistente con el notebook 01 (leptokurtosis generalizada, rechazo de normalidad en el 100% del universo).
3. **Parámetros estáticos:** la media y covarianza históricas pueden cambiar ante shocks macroeconómicos. Se observó, incluso entre corridas del mismo día, que la ventana móvil de descarga (`period="2y"`) desplaza los pesos óptimos cuando cambian los precios intradía más recientes.

### Herramientas

`pandas` · `numpy` · `yfinance` · `scipy.optimize` · `matplotlib`

---

---

## 03 — Métricas de Riesgo: VaR, CVaR y Drawdown

**Notebook:** `03_riesgo.ipynb`  
**Período:** Julio 2024 – Julio 2026  
**Universo:** 9 acciones del IPSA — mismo universo de NB01 y NB02  
**Benchmark:** ECH (iShares MSCI Chile ETF)

### Metodología

- Descarga de precios históricos vía `yfinance` y cálculo de retornos diarios
- **VaR Histórico:** percentil empírico al 5% y 1% de la distribución de retornos. Sin supuesto distribucional.
- **VaR Paramétrico:** `μ − z·σ`, con z = 1.645 (95%) y z = 2.326 (99%). Asume normalidad.
- **VaR Monte Carlo:** 10.000 retornos simulados desde distribución normal calibrada con parámetros históricos. Semilla fija (`np.random.seed(42)`) para reproducibilidad.
- **CVaR / Expected Shortfall:** promedio de retornos peores que el umbral VaR al 95% y 99%.
- **Maximum Drawdown:** `(precio − cummax) / cummax` como curva temporal completa. Tabla resumen con MDD% por activo.
- **Riesgo de portafolio:** retornos de cada portafolio NB02 como suma ponderada `Σ wᵢ · rᵢ`. Pesos cargados desde CSV exportado en NB02 para garantizar consistencia exacta.
- **Los peores días:** top 10 caídas del portafolio de máximo Sharpe con retorno simultáneo del ECH.

### Resultados

**VaR individual — comparación de métodos (95%):**

| Activo | VaR Histórico | VaR Paramétrico | Divergencia |
|---|---|---|---|
| AGUAS-A.SN | -1.88% | -1.96% | +0.08 pp |
| BCI.SN | -2.42% | -2.52% | +0.10 pp |
| CAP.SN | -2.68% | -2.74% | +0.06 pp |
| CENCOSUD.SN | -2.84% | -2.76% | −0.08 pp |
| CMPC.SN | -3.02% | -2.96% | −0.06 pp |

La divergencia histórico–paramétrico es modesta al 95% y se amplifica en el percentil 99%, donde la leptokurtosis documentada en NB01 tiene mayor peso. Los tres métodos convergen en el rango central de la distribución y divergen en las colas.

**Maximum Drawdown por activo:**

| Activo | MDD |
|---|---|
| CMPC.SN | -39.9% |
| CENCOSUD.SN | -38.2% |
| CAP.SN | -27.6% |
| COPEC.SN | -24.9% |
| FALABELLA.SN | -20.9% |
| ECH (benchmark) | -19.7% |
| AGUAS-A.SN | -18.0% |
| BCI.SN | -16.5% |
| ENELCHILE.SN | -13.2% |
| BSANTANDER.SN | -12.6% |

**Comparación de portafolios NB02 — riesgo y retorno:**

| Portafolio | Sharpe | VaR 95% | CVaR 95% | VaR 99% | CVaR 99% | MDD |
|---|---|---|---|---|---|---|
| Máximo Sharpe | 1.91 | -1.73% | -2.22% | -2.50% | -3.16% | -16.3% |
| Mínima Varianza | 1.01 | -1.34% | -1.99% | -2.57% | -3.12% | -21.6% |
| Pesos iguales | 0.94 | -1.56% | -2.09% | -2.56% | -3.15% | -15.9% |
| ECH (benchmark) | 0.84 | -2.38% | -3.39% | -3.95% | -4.75% | -19.7% |

**Hallazgos clave:**

1. **Beneficio de diversificación:** el VaR 95% promedio de los activos individuales es −2.45%; el portafolio de mínima varianza alcanza −1.34%, una reducción de ~45%. La diversificación intersectorial reduce el riesgo de forma significativa.

2. **Límite de la optimización cuadrática — ahora también visible en drawdown:** la mínima varianza domina en VaR y CVaR (95% y 99%), pero tiene el **peor MDD de los cuatro** (-21.6%), incluso más profundo que el benchmark ECH (-19.7%). Minimizar la varianza (segundo momento) reduce la dispersión diaria pero no protege contra una caída sostenida acumulada en el tiempo — el drawdown mide trayectoria, no dispersión puntual.

3. **Máximo Sharpe como elección dominante:** domina las seis métricas de la tabla simultáneamente, con una ratio retorno-riesgo ~1.9x superior a la mínima varianza (1.91 vs 1.01).

4. **Peores días mayoritariamente sistémicos:** los 10 peores días del portafolio coinciden con caídas del ECH, lo que indica que los episodios extremos son shocks de mercado global, no idiosincrásicos del portafolio chileno.

### Limitaciones

1. **VaR histórico:** asume que el pasado reciente representa el futuro. Una ventana de dos años puede no capturar regímenes de estrés no observados.
2. **Covarianza estática:** los pesos de NB02 se calcularon con correlaciones históricas fijas. En crisis, las correlaciones tienden a converger, reduciendo el beneficio de diversificación cuando más se necesita.
3. **Sensibilidad a la ventana de datos:** al comparar esta corrida con una anterior, el ranking de Sharpe y de MDD entre portafolios cambió de forma no trivial (mínima varianza pasó de superar a ECH en MDD a quedar por debajo). El pipeline usa una ventana móvil (`period="2y"` terminada "hoy"), por lo que resultados exactos varían entre ejecuciones en fechas distintas.
4. **Sin out-of-sample:** calibración y evaluación sobre el mismo período. NB05 (GARCH) introducirá VaR dinámico; NB06 introducirá validación walk-forward.

### Herramientas

`pandas` · `numpy` · `yfinance` · `matplotlib` · `seaborn` · `scipy.stats`

---

*Próximo notebook (NB04): riesgo sistemático y beta — descomposición de los retornos del portafolio en componente de mercado (β) y alpha residual, mediante regresión OLS contra el benchmark ECH.*
