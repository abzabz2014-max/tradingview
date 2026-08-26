# XAUUSD Market Structure Analyzer

A non-repainting, analysis-only TradingView indicator for **Gold / U.S. Dollar** written in Pine Script v6. It combines a confirmed 1-hour trend bias, local trend, swing structure, key zones, candle action, and a configurable confluence score. It never submits orders.

## Installation

1. Open an XAUUSD chart in TradingView and select **Pine Editor**.
2. Copy all of [`xauusd_analyzer.pine`](xauusd_analyzer.pine) into the editor.
3. Choose **Save**, then **Add to chart**.
4. Use a standard candlestick chart and create alerts only after adjusting the inputs for the broker's XAUUSD feed.

## Recommended workflow

- Start on **1H** to establish context and review the major confirmed swings.
- Use **4H** for broader structure, **15m** for setup development, and **5m** for detailed confirmation.
- The dashboard's **1H bias** is always calculated from the last fully closed hourly candle. On 4H it remains a 1H reference rather than a higher chart timeframe.
- The intended chart timeframes are 5m, 15m, 1H, and 4H. Other timeframes are not blocked, but their behavior has not been tuned.
- Treat every indication as analysis, not a trade recommendation. Combine it with risk controls, session context, and independent judgment.

## What is displayed

| Element | Meaning |
| --- | --- |
| Current trend | Fast/slow EMA direction on the chart timeframe; small EMA separation relative to ATR is **Sideways**. |
| 1H bias | Bullish, bearish, or sideways direction from closed 1H EMA and ATR values. |
| Support / resistance | Most recently confirmed swing low/high, drawn as extending lines. |
| Swing labels | A confirmed pivot classified as Higher High, Lower High, Higher Low, or Lower Low. |
| Demand / supply | ATR-width areas around the newest swing low/high. These are structural areas, not institutional-order guarantees. |
| Liquidity areas | Narrow bands around the newest swing extremes where stops may cluster. |
| BOS | **Break of Structure**: a confirmed close through a swing in the direction of the current/neutral structure. |
| CHoCH | **Change of Character**: a confirmed close through the opposing swing, flipping the tracked structure direction. It is an early shift indication, not proof of reversal. |
| BULL/BEAR CONF | A new scenario meeting both the mandatory setup rules and minimum score. |
| BULL/BEAR INV | An active bullish scenario closed below the latest swing low, or an active bearish scenario closed above the latest swing high. |

Pivots require bars on both sides. A label therefore appears `pivot strength` bars after the actual swing, at the swing bar. This delay is intentional and prevents future-looking signals.

## Confirmation score and scenarios

Bullish and bearish scenarios are independent. Each score ranges from **0 to 12**. The weighting favors directional and structural evidence over contextual touches:

1. closed 1H bias (2 points);
2. local EMA direction (1 point);
3. tracked market structure direction (2 points);
4. a recent breakout (1 point, within the retest window);
5. the first retest that closes back on the valid side of the stored broken level (2 points);
6. interaction with swing liquidity (1 point);
7. interaction with demand (bullish) or supply (bearish) (1 point); and
8. candle confirmation (2 points: a bullish close above the prior high or bearish close below the prior low).

Reaching the score alone is insufficient. Confirmation also requires the closed 1H bias, tracked structure, candle-close trigger, and breakout direction to agree. **Require breakout retest** is enabled by default, so a scenario waits for the first valid retest within the configurable window; disabling it permits confirmation on the breakout close. Each broken pivot and retest can trigger only once. Set **Minimum confirmation score** from 5 through 12; higher values are more selective. A scenario stays active until its structural invalidation occurs.

## Alerts

Six alert conditions are available from TradingView's **Create Alert** dialog:

- Bullish scenario confirmed
- Bearish scenario confirmed
- Bullish scenario invalidated
- Bearish scenario invalidated
- Break of Structure (BOS), for either direction
- Change of Character (CHoCH), for either direction

Choose **Once Per Bar Close**. The script already gates event logic with confirmed bars, but this alert setting supplies an additional operational safeguard. Messages include TradingView's interval and close placeholders.

## Non-repainting design

- Structure and scenario events execute only on confirmed chart candles.
- Swing pivots are used only after their right-side confirmation bars close.
- 1H `request.security()` values use a one-bar offset with `lookahead_on`, the standard pattern for stable, last-confirmed higher-timeframe values.
- No negative offsets, future data, or intrabar order logic is used.
- Historical signals should therefore remain fixed. Live feed differences and revised broker history can still affect source candles.

## Settings and performance

EMA lengths and ATR sideways tolerance control trend classification. Pivot strength controls swing sensitivity. ATR multipliers control zone widths, while the retest window controls how long a breakout remains eligible. Visual layers and labels can be disabled independently.

The indicator reuses two line objects and four box objects instead of creating zones every bar. Swing labels are the only accumulating objects, and TradingView retains them within the declared 100-label budget.

## Known limitations

- Pine cannot verify that the symbol is XAUUSD; the script can be applied elsewhere, but defaults are designed for gold.
- Feed, spread, session, and broker differences can change pivots and signals.
- On a 4H chart, hourly data requested through `request.security()` is sampled into 4H chart bars; manually compare the dashboard against a 1H chart for the intended broker feed.
- Supply, demand, and liquidity are objective ATR bands around confirmed pivots, not order-book data.
- Pivot confirmation introduces deliberate lag. Fast moves may break a level before a new pivot is confirmed.
- TradingView's compiler and live alert creation must be checked manually after pasting because this repository has no official offline Pine v6 compiler.
- This is not a strategy, does not backtest fills, and must not be treated as financial advice.
