# RiskGuard Intraday BTC v1

Estrategia Pine Script v6 para **BTCUSDT @ 15m**. Sistema de 7 filtros
encadenados orientado a **alta calidad de señal y baja frecuencia**: cada
operación debe pasar los siete filtros simultáneamente o no se dispara.

Diseño espejo: las condiciones LONG y SHORT son simétricas. Una sola posición
abierta a la vez, sin pirámide, sin trailing.

## Sistema de 7 Filtros

| # | Filtro          | Lógica LONG                                                             | Lógica SHORT (espejo)                                                   |
|---|-----------------|-------------------------------------------------------------------------|-------------------------------------------------------------------------|
| 1 | Tendencia HTF   | EMA200 en 1h ascendente y precio sobre ella                             | EMA200 en 1h descendente y precio bajo ella                             |
| 2 | Estructura 15m  | Precio sobre EMA50                                                      | Precio bajo EMA50                                                       |
| 3 | Pullback        | Vela toca EMA21 y cierra **alcista** con cuerpo ≥ 40 % del rango        | Vela toca EMA21 y cierra **bajista** con cuerpo ≥ 40 % del rango        |
| 4 | RSI(14)         | 45 ≤ RSI ≤ 70                                                           | 30 ≤ RSI ≤ 55                                                           |
| 5 | ATR(14)         | ATR(14) > SMA(ATR, 50) — volatilidad en expansión                       | ATR(14) > SMA(ATR, 50) — volatilidad en expansión                       |
| 6 | KDJ (9, 3, 3)   | K cruza al alza sobre D en zona K < 50, en las últimas 3 velas          | K cruza a la baja bajo D en zona K > 50, en las últimas 3 velas         |
| 7 | S/R asimétrico  | Soporte (pivotLow 10/5) a **< 2 × ATR** y resistencia a **> 3 × ATR**   | Resistencia (pivotHigh 10/5) a **< 2 × ATR** y soporte a **> 3 × ATR**  |

Todos los pivots usan `ta.pivothigh` / `ta.pivotlow` con `pivotLeft = 10` y
`pivotRight = 5` (se confirman 5 velas después del pivote).

## Gestión de Salida

| Parámetro       | Valor          |
|-----------------|----------------|
| Stop Loss       | `1.5 × ATR(14)` |
| Take Profit     | `3.0 × ATR(14)` |
| Ratio R : R     | `1 : 2`         |
| Trailing        | No              |
| Breakeven       | No              |
| Pirámide        | No (`pyramiding = 0`) |

SL y TP se calculan en el momento de la entrada usando el ATR de la vela que
disparó la señal, y se envían como órdenes `stop` y `limit` con
`strategy.exit()`. La estrategia opera con `process_orders_on_close = true`.

## Uso en TradingView

1. Abre TradingView → **Pine Editor**.
2. Pega el contenido de [`riskguard_intraday_btc_v1.pine`](./riskguard_intraday_btc_v1.pine).
3. **Save** → **Add to chart**.
4. Configura el gráfico en **BTCUSDT** y temporalidad **15m**.
5. Los inputs vienen agrupados por sección (filtros 1 – 7, salida, alertas).
   Los valores por defecto coinciden con la especificación de este README.
6. Para alertas:
   - Clic derecho en el gráfico → **Add alert…**
   - **Condition** = la estrategia "RiskGuard Intraday BTC v1".
   - **Message** = `{{strategy.order.alert_message}}`
   - **Frequency** = *Once per bar close*.
   - **Webhook URL** = tu endpoint (Telegram bot relay, exchange bridge, etc.).

## Formato de Alerta (Webhook)

El cuerpo enviado al webhook es JSON plano:

```json
{
  "symbol":   "BTCUSDT",
  "side":     "buy",
  "price":    50000.00,
  "sl":       49250.00,
  "tp":       51500.00,
  "tf":       "15m",
  "strategy": "riskguard_intraday_v1"
}
```

| Campo      | Tipo    | Notas                                          |
|------------|---------|------------------------------------------------|
| `symbol`   | string  | Configurable en inputs (default `BTCUSDT`).    |
| `side`     | string  | `"buy"` para LONG, `"sell"` para SHORT.        |
| `price`    | number  | Precio de cierre de la vela disparadora.       |
| `sl`       | number  | Stop loss absoluto.                            |
| `tp`       | number  | Take profit absoluto.                          |
| `tf`       | string  | Etiqueta de TF (default `"15m"`).              |
| `strategy` | string  | Identificador fijo `"riskguard_intraday_v1"`.  |

## Criterios Mínimos de Validación

Antes de operar con capital real, el sistema debe pasar un backtest sobre al
menos **12 meses** de histórico BTCUSDT 15m cumpliendo **todos** estos umbrales:

| Métrica         | Umbral mínimo |
|-----------------|---------------|
| Nº de trades    | ≥ 100         |
| Profit Factor   | > 1.5         |
| Drawdown máx.   | < 15 %        |
| Win rate        | > 40 %        |

Si una sola métrica no se cumple → re-tunear inputs dentro de rangos razonables
o descartar el setup. **No** desplegar capital real con un backtest fallido.

## Disclaimer

Este código se publica con fines **educativos y de investigación**. No
constituye asesoramiento financiero ni recomendación de inversión. Los mercados
de criptoactivos son altamente volátiles y puedes perder la totalidad del
capital invertido. El backtesting no garantiza resultados futuros: condiciones
de mercado, slippage, fees, cambios de régimen y errores de ejecución pueden
deteriorar el rendimiento real respecto al simulado. Opera únicamente bajo tu
propia responsabilidad y con capital cuya pérdida puedas asumir.
