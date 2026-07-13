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
**Período:** Julio 2024 – Julio 2026 · **Frecuencia:** Diaria · **Fuente:** Yahoo Finance API

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
| AGUAS-A.SN | -1.88% | -1.96% | -1.98% |
| BCI.SN | -2.42% | -2.52% | -2.46% |
| BSANTANDER.SN | -1.93% | -1.99% | -1.96% |
| CAP.SN | -2.68% | -2.74% | -2.75% |
| CENCOSUD.SN | -2.84% | -2.76% | -2.78% |
| CMPC.SN | -3.02% | -2.96% | -3.01% |
| COPEC.SN | -2.40% | -2.49% | -2.45% |
| ENELCHILE.SN | -2.19% | -2.25% | -2.25% |
| FALABELLA.SN | -2.71% | -2.70% | -2.73% |

La divergencia histórico–paramétrico es modesta al 95% y se amplifica al 99%, precisamente donde la kurtosis importa. AGUAS-A y CAP presentan las brechas más pronunciadas, consistente con su alto exceso de kurtosis en NB01.

### Maximum Drawdown por activo

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

### Comparación de portafolios — riesgo y retorno

| Portafolio | Sharpe | VaR 95% | CVaR 95% | VaR 99% | CVaR 99% | MDD |
|---|---|---|---|---|---|---|
| Máximo Sharpe | 1.91 | -1.73% | -2.22% | -2.50% | -3.16% | -16.3% |
| Mínima Varianza | 1.01 | -1.34% | -1.99% | -2.57% | -3.12% | -21.6% |
| Pesos iguales | 0.94 | -1.56% | -2.09% | -2.56% | -3.15% | -15.9% |
| ECH (benchmark) | 0.84 | -2.38% | -3.39% | -3.95% | -4.75% | -19.7% |

La mínima varianza domina en VaR y CVaR (95% y 99%), pero **pierde esa ventaja en drawdown**: con esta ventana de datos su MDD (-21.6%) es el peor de los cuatro portafolios, incluso más profundo que el del benchmark pasivo ECH (-19.7%). Minimizar la varianza (segundo momento) reduce la dispersión diaria pero no protege contra una caída sostenida que se acumula a lo largo de meses — el drawdown mide trayectoria, no dispersión puntual. El máximo Sharpe domina las seis métricas simultáneamente y entrega una relación retorno-riesgo ~1.9x superior a la mínima varianza (1.91 vs 1.01).

*Nota: en la corrida anterior de este proyecto, la mínima varianza sí superaba a ECH en las tres métricas de riesgo. El cambio de ranking en MDD ilustra la sensibilidad de estos resultados al punto exacto de corte de los datos — ver Limitaciones.*

### Los peores días — portafolio de máximo Sharpe

Los diez peores días concentran pérdidas entre -2.2% y -4.5%, con alta coincidencia con el ECH, lo que indica que la mayoría de los episodios extremos son sistémicos (shocks de mercado global) más que idiosincrásicos del portafolio.

## Limitaciones

1. **VaR histórico:** asume que el pasado reciente es representativo del futuro. Una ventana de dos años puede no capturar regímenes de estrés no observados.
2. **Covarianza estática:** los pesos de NB02 usan correlaciones históricas fijas. En crisis, las correlaciones tienden a converger, deteriorando la diversificación cuando más se necesita.
3. **Sensibilidad a la ventana de datos:** al comparar esta corrida con la anterior, el ranking de Sharpe y de MDD entre portafolios cambió de forma no trivial (mínima varianza pasó de superar a ECH en MDD a quedar por debajo). El pipeline descarga datos con una ventana móvil (`period="2y"` terminada "hoy"), por lo que cada ejecución en un día distinto —o incluso el mismo día si el mercado sigue abierto— puede producir resultados algo distintos.
4. **Sin out-of-sample:** el mismo período se usa para calibrar y evaluar el riesgo. NB05 (GARCH) abordará el VaR dinámico; NB06 introducirá validación walk-forward.

## Herramientas

`pandas` · `numpy` · `yfinance` · `matplotlib` · `seaborn` · `scipy.stats`
