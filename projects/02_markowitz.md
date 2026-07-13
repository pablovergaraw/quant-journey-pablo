# Proyecto 02 — Optimización de Portafolio: Modelo de Markowitz

## Objetivo

Construir un modelo de optimización de portafolio basado en la Teoría Moderna de Portafolios (Markowitz, 1952) aplicado a las acciones del IPSA analizadas en el Proyecto 01. El objetivo es identificar la combinación de pesos óptima que maximiza el retorno ajustado por riesgo (Sharpe ratio) y trazar la frontera eficiente completa mediante dos enfoques: simulación de Monte Carlo y optimización analítica con `scipy`.

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

1. Cálculo de retornos diarios, vector de retornos medios y matriz de covarianza
2. **Simulación de Monte Carlo:** 10.000 portafolios con pesos aleatorios normalizados; nube riesgo-retorno coloreada por Sharpe ratio
3. **Optimización analítica — máximo Sharpe:** `scipy.optimize.minimize` minimizando el Sharpe negativo, con restricciones de suma unitaria y sin posiciones cortas
4. **Optimización analítica — mínima varianza:** minimización de la volatilidad sin restricción de retorno
5. **Frontera eficiente analítica:** minimización de volatilidad para 50 niveles de retorno objetivo, filtrando la parte ineficiente por pendiente
6. **Capital Market Line (CML):** recta desde la TPM (~5.5% anual) tangente a la frontera en el portafolio de máximo Sharpe
7. Anualización de métricas (retorno × 252, volatilidad × √252) y cálculo de Sharpe real con TPM como tasa libre de riesgo
8. Exportación de los tres portafolios a CSV para uso en el Proyecto 03

## Resultados

### Portafolio óptimo (máximo Sharpe)

| Activo | Peso |
|---|---|
| BCI | ~37% |
| Falabella | ~26% |
| BancoSantander | ~16% |
| Aguas Andinas | ~12% |
| Enel Chile | ~9% |
| CAP, CMPC, COPEC, Cencosud | 0% |

**Métricas anualizadas:**

| Métrica | Valor |
|---|---|
| Sharpe ratio sin rf (anualizado) | ~2.09 |
| Sharpe ratio real (rf = 5.5%) | ~1.78 |
| Supera benchmark ECH en Sharpe | ~2.19x |

### Portafolio de mínima varianza

| Activo | Peso |
|---|---|
| Aguas Andinas | ~34% |
| BancoSantander | ~16% |
| Enel Chile | ~16% |
| CAP | ~15% |
| Cencosud | ~8% |
| Copec | ~8% |
| BCI, Falabella | <4% |

El portafolio de mínima varianza concentra en AGUAS-A por ser el activo menos volátil del universo, diversificando el resto entre activos con baja covarianza. Sacrifica retorno esperado a cambio de minimizar la volatilidad total.

### Capital Market Line

Combinar el portafolio de máximo Sharpe con el activo libre de riesgo domina a cualquier portafolio puro sobre la frontera eficiente para el mismo nivel de riesgo — valida el rol del portafolio óptimo como único portafolio riesgoso relevante en presencia de un activo libre de riesgo.

## Limitaciones

1. **Overfitting histórico:** el portafolio se calibra y evalúa sobre el mismo período. Los pesos óptimos pasados no garantizan rendimiento futuro.
2. **Supuesto de normalidad:** Markowitz asume distribución normal de retornos, subestimando eventos extremos — inconsistente con el Proyecto 01 (leptokurtosis generalizada, rechazo de normalidad en el 100% del universo).
3. **Parámetros estáticos:** la media y covarianza históricas pueden cambiar ante shocks macroeconómicos. Se observó, incluso entre corridas del mismo día, que usar una ventana móvil (`period="2y"` terminada "hoy") desplaza los pesos óptimos cuando cambian los precios intradía más recientes — el modelo es tan estable como los datos que recibe.

## Herramientas

`pandas` · `numpy` · `yfinance` · `scipy.optimize` · `matplotlib`
