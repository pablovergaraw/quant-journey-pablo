# Proyecto 03 — Métricas de Riesgo: VaR, CVaR y Drawdown

## Objetivo

Cuantificar el riesgo del universo de acciones chilenas analizado en los proyectos anteriores mediante tres métricas fundamentales de la gestión de riesgo: Value at Risk (VaR), Conditional Value at Risk (CVaR) y Maximum Drawdown. El análisis contrasta tres metodologías de estimación del VaR, evalúa el beneficio de la diversificación y compara el perfil de riesgo de los portafolios optimizados en el Proyecto 02.

Este notebook cierra el ciclo analítico iniciado en NB01 (caracterización estadística) y continuado en NB02 (optimización): los mismos portafolios que maximizan el Sharpe ratio se evalúan ahora desde el punto de vista del riesgo real de pérdida.

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

**Benchmark:** ECH (iShares MSCI Chile ETF)  
**Período:** Mayo 2024 – Mayo 2026 · **Frecuencia:** Diaria · **Fuente:** Yahoo Finance API

## Metodología

### Value at Risk — tres métodos

- **Histórico:** percentil empírico de la distribución de retornos al 5% y 1%. No asume distribución subyacente.
- **Paramétrico:** `μ − z·σ`, con z = 1.645 (95%) y z = 2.326 (99%). Asume normalidad.
- **Monte Carlo:** 10.000 retornos simulados desde una distribución normal calibrada con parámetros históricos; percentil 5% y 1% de la simulación.

La divergencia entre el método histórico y el paramétrico cuantifica directamente el impacto de la leptokurtosis documentada en NB01: donde los retornos tienen colas más gruesas que la normal, el VaR paramétrico subestima el riesgo.

### CVaR / Expected Shortfall

Promedio de los retornos peores que el umbral VaR. Responde: cuando el día malo ocurre, ¿cuánto se pierde en promedio? Siempre más negativo que el VaR correspondiente.

### Maximum Drawdown

Caída máxima desde un peak histórico hasta el valle siguiente, calculada como `(precio − cummax) / cummax`. Se visualiza como curva temporal completa, no solo el mínimo puntual.

### Riesgo de portafolio

Los retornos de cada portafolio (máximo Sharpe, mínima varianza, pesos iguales) se calculan como suma ponderada de retornos individuales. Los pesos se cargan directamente del CSV exportado en NB02 para garantizar consistencia exacta.

## Resultados

### VaR individual — comparación de métodos (95%)

| Activo | VaR Histórico | VaR Paramétrico | VaR Monte Carlo |
|---|---|---|---|
| AGUAS-A.SN | -1.88% | -1.95% | -1.97% |
| BCI.SN | -2.28% | -2.45% | -2.39% |
| BSANTANDER.SN | -1.90% | -1.95% | -1.92% |
| CAP.SN | -2.65% | -2.72% | -2.73% |
| CENCOSUD.SN | -2.79% | -2.70% | -2.72% |
| CMPC.SN | -2.95% | -2.93% | -2.98% |
| COPEC.SN | -2.40% | -2.49% | -2.45% |
| ENELCHILE.SN | -2.16% | -2.24% | -2.24% |
| FALABELLA.SN | -2.67% | -2.63% | -2.66% |

La divergencia histórico–paramétrico es modesta al 95% y se amplifica al 99%, precisamente donde la kurtosis importa. AGUAS-A y CAP presentan las brechas más pronunciadas, consistente con su alto exceso de kurtosis en NB01.

### Maximum Drawdown por activo

| Activo | MDD |
|---|---|
| CMPC.SN | -46.8% |
| CENCOSUD.SN | -37.9% |
| CAP.SN | -34.3% |
| COPEC.SN | -26.5% |
| FALABELLA.SN | -20.9% |
| ECH (benchmark) | -19.7% |
| AGUAS-A.SN | -18.0% |
| BCI.SN | -16.5% |
| ENELCHILE.SN | -13.7% |
| BSANTANDER.SN | -12.6% |

### Comparación de portafolios — riesgo y retorno

| Portafolio | Sharpe | VaR 95% | CVaR 95% | VaR 99% | CVaR 99% | MDD |
|---|---|---|---|---|---|---|
| Máximo Sharpe | 1.78 | -1.67% | -2.26% | -2.46% | -3.26% | -16.2% |
| Mínima Varianza | 0.89 | -1.36% | -1.97% | -2.58% | -3.09% | -16.7% |
| Pesos iguales | 0.73 | -1.55% | -2.07% | -2.56% | -3.15% | -15.5% |
| ECH (benchmark) | 0.72 | -2.36% | -3.39% | -3.95% | -4.75% | -19.7% |

Los tres portafolios superan al benchmark ECH en todas las métricas de riesgo. La mínima varianza domina en VaR 95%, pero esa ventaja se reduce al 99%: la optimización cuadrática minimiza el segundo momento (varianza) pero no protege las colas del cuarto momento (kurtosis). El máximo Sharpe entrega una relación retorno-riesgo más del doble que la mínima varianza (1.78 vs 0.89).

### Los peores días — portafolio de máximo Sharpe

Los diez peores días concentran pérdidas entre -2.2% y -4.7%, con alta coincidencia con el ECH, lo que indica que la mayoría de los episodios extremos son sistémicos (shocks de mercado global) más que idiosincrásicos del portafolio.

## Limitaciones

1. **VaR histórico:** asume que el pasado reciente es representativo del futuro. Una ventana de dos años puede no capturar regímenes de estrés no observados.
2. **Covarianza estática:** los pesos de NB02 usan correlaciones históricas fijas. En crisis, las correlaciones tienden a converger, deteriorando la diversificación cuando más se necesita.
3. **Sin out-of-sample:** el mismo período se usa para calibrar y evaluar el riesgo. NB05 (GARCH) abordará el VaR dinámico; NB06 introducirá validación walk-forward.

## Herramientas

`pandas` · `numpy` · `yfinance` · `matplotlib` · `seaborn` · `scipy.stats`
