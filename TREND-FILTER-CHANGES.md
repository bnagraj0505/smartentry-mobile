# Trend filter patch (applied to the built app)

These changes were made directly in the compiled bundle (`assets/device-trendf04.js`).
Port them into the source before your next build, or the build will overwrite them.

## What changed in the chart signals (pullback + SMC reversal)

1. **Higher-timeframe filter.** The chart's signals are checked against the next timeframe up:
   1m → 15m, 5m → 1h, 15m → 1h, 1h → 1d, 4h → 1d.
   The higher timeframe is an *uptrend* when EMA 20 > EMA 50, close > EMA 50 and EMA 50 is not falling
   over 5 bars; a *downtrend* is the mirror image; anything else is *no clear trend*.
   Only closed higher-timeframe candles are used (no look-ahead).
2. **No trade against the higher timeframe.** Longs are blocked in a higher-timeframe downtrend,
   shorts in an uptrend.
3. **ADX gate for pullbacks.** Pullback signals need ADX(14) ≥ 20 on the chart timeframe.
4. **Reversals (⟲) only with the higher-timeframe trend.** A bullish SMC reversal is taken only when the
   higher timeframe is in an uptrend, a bearish one only in a downtrend. When higher-timeframe data
   is unavailable (1d chart, or the fetch fails), reversals are off.
5. Because opposite signals now have to pass the same filter, a counter-trend reversal can no longer
   close a trade that is running with the trend.
6. The Signals status line now ends with the higher-timeframe state, and three lines were added to
   "The rules".

## Where it lives in the source

- The strategy loop already had a filter hook: `strategy(candles, reversals, filter, now)` called
  `filter(side, time)`. It now calls `filter(side, time, setup, index)`.
- The chart endpoint (the function that returns `signals`, `smc`, `regime`) fetches 500 higher-timeframe
  candles, builds the filter and passes it in instead of `null`.

## Other files

- `index.html` points to the renamed bundle.
- `sw.js` cache name bumped to `smartentry-v2` so phones drop the old cached code.

## Wording: "Premium" zone shown as "Expensive"

The SMC premium/discount zone (top of the recent range) is labelled **Expensive** everywhere on screen:
the chart overlay, the Price Action badge and meter, the "Expensive / discount now" heading, the layer
toggle and the explanation text. Internal names (`premium`, CSS classes) are unchanged, and wording about
an option's price (premium paid, "Entry (premium)") is left as it was.

## CPR (central pivot range) on the Price Action page

- New panel at the top of the Price Action side column, plus a chart layer "CPR (central pivot range)"
  (blue band between TC and BC, pivot dashed), switchable under Layers.
- From the previous trading day (same session logic as the existing "previous day" levels):
  P = (H + L + C) / 3, BC = (H + L) / 2, TC = 2P − BC; the higher of TC/BC is drawn as the top.
- Width as % of price, ranked against the last 20 sessions: narrowest 30% = "Narrow CPR: trending day likely",
  widest 30% = "Wide CPR: slow / sideways day likely" (fixed 0.15% / 0.45% thresholds when history is short).
- Price vs CPR: above = buy bias, below = sell bias, inside = sideways.
- Today's open vs CPR sets the opening bias. Setup: opened above, a candle dips to the top CPR, closes back
  above it and green = buy (stop just below the CPR, target 2R); mirror for opened below. A close through the
  far side of the CPR shows "Bias failed". When the market is closed it shows the CPR for the next session.
- Code: `seCprOf`, `seCpr`, `seCprP` before the price-action analysis function; `cpr` field added to its result;
  `paintCpr` on the chart overlay class.

## Hide / show the right-side panel on Price Action, Setups and Order Flow

- The same ⟩ / ⟨ button as on the Dashboard, sharing the same on/off state. Desktop and landscape layouts
  only (wider than 860px); narrower screens stack the panel under the chart as before.
- Stylesheet renamed to `assets/device-trendf03.css` (rules appended at the end).

## FYERS login from the hosted phone app

- A saved Redirect URL pointing at 127.0.0.1 / localhost is ignored when the app runs from a real web
  address; the page's own address is sent instead. The FYERS tab notes this next to the address to register.

## FYERS login on iPhone (Safari handoff)

- iPhone keeps Home Screen web apps' storage separate from Safari. When the FYERS login returns to Safari,
  Safari has no saved App ID / Secret / login state. Instead of failing, that page now shows a one-time
  login code with a Copy button.
- The FYERS tab in the app has a "Paste login code" button: it checks the code against the pending Connect
  (same state, under 10 minutes old), exchanges it for the access token in the app itself and reloads connected.
- Code: `seExchange`, `seFinish`, `sePaste`, `seHandoff` before the login-return function; the return function
  hands off when this storage has no pending login.
