# Proyecto 01 — Análisis Exploratorio del Mercado Accionario Chileno

## Objetivo

Construir un pipeline de análisis de datos sobre acciones del IPSA (Índice de Precio Selectivo de Acciones), el principal índice bursátil de Chile, con el fin de caracterizar el comportamiento histórico de los activos y establecer la base para modelos de optimización de portafolios.

## Activos analizados

| Ticker | Empresa | Sector |
|---|---|---|
| COPEC.SN | Copec S.A. | Energía |
| FALABELLA.SN | S.A.C.I. Falabella | Retail |
| CMPC.SN | Empresas CMPC S.A. | Forestal / Papel |

**Período:** Mayo 2024 – Mayo 2026 · **Frecuencia:** Diaria · **Fuente:** Yahoo Finance API

## Metodología

1. Extracción de precios históricos OHLCV via API
2. Aislamiento de precios de cierre ajustados
3. Normalización de series para comparación de rendimiento relativo
4. Cálculo de retornos totales y diarios
5. Análisis estadístico descriptivo (media, volatilidad, distribución)
6. Matriz de correlación entre activos

## Resultados

### Retornos totales (Mayo 2024 – Mayo 2026)

| Activo | Retorno |
|---|---|
| FALABELLA.SN | +115.8% |
| COPEC.SN | -11.6% |
| CMPC.SN | -37.2% |

*Análisis estadístico completo en desarrollo.*

## Herramientas

Python 3.13 · pandas 3.0 · yfinance 1.3 · matplotlib · seaborn
