# RSI Extreme Scalper BTC v1

Estrategia Pine Script v6 ultra-mínima para **BTCUSDT @ 15m**. Una sola idea:
entrar contra extremos de RSI buscando un rebote pequeño antes del stop ancho.

## Reglas

| Concepto | Valor |
|----------|-------|
| Entrada LONG  | `RSI(14) < 15` |
| Entrada SHORT | `RSI(14) > 85` |
| Take Profit   | `+0.30 %` desde precio de entrada |
| Stop Loss     | `-1.00 %` desde precio de entrada |
| Ratio R:R     | `0.3 : 1` |
| Posiciones    | Una a la vez (`pyramiding = 0`) |
| Ejecución     | `process_orders_on_close = true` |

Sin filtros de tendencia, sin pullback, sin S/R. RSI extremo y stop ancho.

## Matemáticas a priori

Con TP 0.30 % y SL 1.00 %, el win rate de breakeven **bruto** es:

```
BE_WR_gross = SL / (TP + SL) = 1.00 / 1.30 = 76.9 %
```

Añadiendo comisión 0.04 % × 2 (round-trip) = 0.08 % por trade:

```
BE_WR_net ≈ 1.08 / (0.22 + 1.08) ≈ 83.1 %
```

Es decir, sin >83 % de aciertos esto pierde dinero por construcción.

## Uso en TradingView

1. Pine Editor → pegar `rsi_extreme_btc_v1.pine`.
2. Save → Add to chart.
3. Cargar BTCUSDT en temporalidad 15m.
4. Backtest → "Informe de estrategia".

## Webhook

```json
{
  "symbol":   "BTCUSDT",
  "side":     "buy",
  "price":    50000.00,
  "sl":       49500.00,
  "tp":       50150.00,
  "tf":       "15m",
  "strategy": "rsi_extreme_btc_v1"
}
```

## Disclaimer

Código educativo. No es asesoramiento financiero. El backtesting no garantiza
resultados futuros. Opera bajo tu propia responsabilidad.
