# Proyecto 02 — Optimización de Portafolio: Modelo de Markowitz

## Objetivo

Construir un modelo de optimización de portafolio basado en la Teoría Moderna de Portafolios (Markowitz, 1952) aplicado a las acciones del IPSA analizadas en el Proyecto 01. El objetivo es identificar la combinación de pesos óptima que maximiza el retorno ajustado por riesgo (Sharpe ratio) y visualizar la frontera eficiente mediante simulación de Monte Carlo.

## Activos analizados

| Ticker | Empresa | Sector |
|---|---|---|
| COPEC.SN | Copec S.A. | Energía |
| FALABELLA.SN | S.A.C.I. Falabella | Retail |
| CMPC.SN | Empresas CMPC S.A. | Forestal / Papel |

**Período:** Mayo 2024 – Mayo 2026 · **Frecuencia:** Diaria · **Fuente:** Yahoo Finance API

## Metodología

1. Cálculo de retornos diarios y matriz de covarianza
2. Definición de función de evaluación de portafolio: retorno esperado, volatilidad y Sharpe
3. Simulación de Monte Carlo: 10.000 portafolios con pesos aleatorios normalizados
4. Identificación del portafolio de máximo Sharpe
5. Visualización de la nube de portafolios (frontera eficiente aproximada)

### Cálculo de métricas por portafolio

- **Retorno esperado:** producto punto entre vector de pesos y retornos medios diarios
- **Varianza:** `w^T · Cov · w` (producto matricial)
- **Volatilidad:** raíz cuadrada de la varianza
- **Sharpe simplificado:** retorno / volatilidad (sin deducción de tasa libre de riesgo)

## Resultados

### Portafolio de máximo Sharpe (simulación Monte Carlo)

| Métrica | Valor |
|---|---|
| Sharpe ratio | 0.100 |
| Retorno diario esperado | +0.164% |
| Volatilidad diaria | 1.65% |

### Distribución de pesos óptimos

| Activo | Peso |
|---|---|
| CMPC.SN | ~0% |
| COPEC.SN | ~3% |
| FALABELLA.SN | ~97% |

La concentración en Falabella refleja su desempeño superior en el período analizado (+115.8% total). Este resultado ilustra la principal limitación del modelo: la optimización histórica tiende a concentrar el capital en el activo con mejor desempeño pasado, lo que no garantiza resultados futuros equivalentes.

## Limitaciones del modelo

- Asume que los retornos históricos son representativos de los retornos futuros
- No incluye tasa libre de riesgo en el cálculo del Sharpe
- No permite venta corta (pesos restringidos al intervalo [0, 1])
- La frontera eficiente se aproxima por simulación, no se calcula analíticamente

## Herramientas

Python 3.13 · numpy · pandas · yfinance · matplotlib
