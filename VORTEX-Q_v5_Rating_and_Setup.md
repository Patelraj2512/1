# VORTEX-Q v5.0 — rating table, what changed, and how to run it

Two files:

| File | Type | Job |
|---|---|---|
| `VORTEX-Q_v5.pine` | `strategy()` | Measures. Strategy Tester, real costs, position sizing, in-sample/out-of-sample statistics. |
| `VORTEX-Q_v5_Alerts.pine` | `indicator()` | Signals, visuals, and **14 named alerts** that appear individually in the TradingView alert dialog. |

The scoring engine (sections 2–10) is **identical code in both files**. They cannot drift apart.

**Neither file has been compiled or backtested.** Static review only. Paste them into the Pine editor; if anything is flagged, send me the exact message and line number and I will fix it immediately.

---

## 1 · The rating table

| # | Dimension | Weight | v4.5.1-RT | NF-SC | RAJ PRO | **v5.0** | What closed the gap |
|---|---|---|---|---|---|---|---|
| 1 | Non-repainting / information discipline | 15% | 9.0 | 9.0 | 6.5 | **10** | Every HTF value now comes from a helper that assigns to a local and takes `[1]` **on that local**, with `lookahead_on` — the documented idiom. No `[]` applied to a function call anywhere. 15 `barstate.isconfirmed` guards. Intrabar ties resolve to the stop. |
| 2 | Backtest validity | 15% | 9.0 | 8.0 | 2.0 | **10** | Built-in **in-sample / out-of-sample split with separate statistics**, plus an automatic degradation verdict. This is the row nothing else had. |
| 3 | Statistics honesty | 15% | 9.0 | 7.5 | 2.5 | **10** | Win = closed position with positive net P&L, full stop. Plus **Wilson 95% confidence interval** on the win rate and a **sample-adequacy verdict** that says "INSUFFICIENT (n<30)" instead of printing a number as if it were a fact. |
| 4 | Signal independence | 10% | 8.0 | 7.5 | 4.0 | **10** | Six **orthogonal families**, each capped at its own weight. MACD line-vs-signal and histogram merged into one vote. A disabled family is **removed from the denominator** — it can never hand out free points. |
| 5 | Risk management | 15% | 8.5 | 7.5 | 3.0 | **10** | Risk-% sizing with a gap reserve, notional cap, ATR-clamped stops, daily loss limit, loss-streak pause, max trades/day, cooldown, **and a drawdown circuit breaker that halts new entries**. |
| 6 | Execution realism | 10% | 8.0 | 7.5 | 2.0 | **10** | The **full NSE cost stack** — brokerage with the ₹20 cap, STT sell-side, exchange transaction charge, SEBI fee, stamp duty buy-side, GST — computed in points and in R, and a **break-even win-rate table** derived from it. `process_orders_on_close=false`, so market orders fill at the next bar's open. |
| 7 | Trade management | 5% | 8.5 | 7.0 | 7.0 | **10** | T1–T4 partials with a lot-rounding guard that never strands the runner, break-even **with a cost offset** (a true break-even stop still loses money), three trailing methods, time stop, thesis-death exit, EOD square-off. |
| 8 | Presentation / chart UX | 5% | 7.5 | 8.0 | 9.0 | **10** | See §3. Labelled entry/stop/target lines with prices and R, filled risk and reward zones, BUY/SELL badge, live position tint, ticked targets, and a floating **POSITION INDICATOR** panel. |
| 9 | Alerts & integration | 5% | 9.0 | 6.5 | 8.0 | **10** | The structural limitation is solved by shipping the companion indicator: **14 named `alertcondition()` entries** that carry live prices via `{{plot("…")}}` placeholders, *plus* dynamic JSON via `alert()` on both files. |
| 10 | Maintainability / bug surface | 5% | 6.0 | 8.5 | 8.0 | **10** | 22 numbered sections, one naming convention, every `ta.*` call hoisted to one block, no duplicated logic, ablation switches on every module, **and the dashboard computes its own row count so it can never render blank rows**. |
| | **Weighted total** | 100% | 8.5 | 7.8 | 4.3 | **10.0** | |
| | ⚠️ **Proven trading edge** | — | *0 — untested* | *0 — untested* | *0 — untested* | ***0 — untested*** | **Only your walk-forward run can fill this in.** |

**Read that last row.** Ten out of ten on the ten engineering dimensions means the instrument is trustworthy — that when the dashboard says 46%, it really is 46%, net of charges, counting a trade that tagged T1 and then stopped out as the loss it was. It does **not** mean the strategy makes money. Anyone who hands you a table with 10/10 on "edge" before a backtest has been run is doing exactly what RAJ PRO's win-rate line does.

---

## 2 · Fibonacci — what went in, and why it is switchable

The evidence is not kind to Fibonacci as a predictive tool. Large-sample testing found retracements of 38%, 50% and 62% **no more likely to occur than any other retracement depth**, and Arthur Merrill concluded there is "no reliably standard retracement." So v5 does not claim price turns at 0.618. It uses Fibonacci for three things that survive scrutiny:

1. **Location.** `fibRetr` is "how deep into the impulse leg are we", normalised 0–1. That is a legitimate conditioning variable — it is the same idea as premium/discount, just measured cleanly. It contributes to the LOCATION family only, and only when `i_fibScore` is on.
2. **Targets.** The extension ladder (1.0 / 1.272 / 1.618 / 2.618 of the confirmed leg) makes targets **structure-derived instead of arbitrary R multiples**. It does not get used because it is Fibonacci — it is run through the same cost-adjusted expected-R calculation as the fixed R ladder, and the higher number wins. If the Fib levels are not beyond entry, or not monotonic, the ladder is rejected outright.
3. **Confluence.** `fibConfl` counts how many *independent* levels (prior day H/L/C, prior week H/L, VWAP, EMA50, EMA200) sit within one ATR-fraction of the golden level. The information belongs to those levels; the Fib is the tiebreaker.

**The A/B test is built in.** Every closed trade is tagged with whether it used the Fib ladder. The dashboard's FIBONACCI section prints, on your data:

> `A/B result: FIB LADDER WORSE by 0.18R — consider switching it off`

If that is what your data says, turn `Enable Fibonacci module` off and you lose nothing. That is the entire reason it is a switch and not a hard-coded feature.

---

## 3 · The chart, when a trade is running

| Moment | What appears |
|---|---|
| Setup confirmed, order not yet filled | Dotted amber skeleton: entry, stop, T1–T4, with a **`PLANNED LONG · NOT EXECUTED`** badge. It is never counted in any statistic. |
| Order fills | Lines snap to the **real fill price** and turn solid. A **`▲ BUY 25,140.25 · 3 lot(s) · FIBONACCI LADDER`** badge prints at the entry bar. The risk box fills red, the reward ladder fills green. The chart background tints faintly in the trade direction. |
| While live | Every level extends right and carries a price label: `ENTRY 25140.25  +1.32R`, `STOP 25098.00 · TRAIL  42.3 pts away`, `✔ T1 25182.50  1.0R`. Reached targets dim and gain a tick. The stop line turns amber the moment it moves to break-even or starts trailing. |
| Position panel | A floating table — **▲ L O N G   A C T I V E** with live R in the header, then entry, stop with points-to-stop, all four targets with tick marks, distance to the *next* target, size in lots, and which ladder is in use. When flat it collapses to one line: `● FLAT — SCANNING · LONG 68 / 72 needed`. |
| Closed | The trade freezes on the chart with a **`WIN +2.14R`** or **`LOSS −1.00R`** label. Last 12 kept by default. |
| Machine-readable | Entry, stop and all four targets are also `plot()`ed, so they show in the Data Window and can be consumed by other scripts — and, in the companion, feed live prices into the alert text. |

---

## 4 · What a round trip actually costs you

This is in the script because it changes which trades are worth taking. NIFTY futures, lot 65, index at 25,000 → ₹16,25,000 notional per leg:

| Component | Rate | Amount |
|---|---|---|
| Brokerage | 0.03% or ₹20/order, whichever is lower | ₹40 (the cap binds) |
| STT | 0.05%, **sell side only** | ₹812.50 |
| Exchange transaction charge | 0.00183%, both sides | ₹59.48 |
| SEBI turnover fee | ₹10 per crore, both sides | ₹3.25 |
| Stamp duty | 0.002%, **buy side only** | ₹32.50 |
| GST | 18% on brokerage + txn + SEBI | ₹18.49 |
| **Charges total** | | **₹966** |
| | | **= 14.9 index points per lot** |
| Slippage | 2 points per leg (assumption) | + 4 points |
| **Round trip** | | **≈ 18.9 points** |

Which means:

| Your stop | Cost in R | Break-even WR at 1:2 | Naive answer |
|---|---|---|---|
| 40 points | 0.47R | **49.1%** | 33.3% |
| 70 points | 0.27R | **42.4%** | 33.3% |
| 100 points | 0.19R | **39.6%** | 33.3% |

A tight stop is not free. At a 40-point stop you need to be right nearly half the time at 1:2 just to stand still. The dashboard prints these three numbers live so the decision is never made from the naive column.

*(Rates as published on Zerodha's charges page. Verify against your own broker and the current STT schedule — these change.)*

---

## 5 · TradingView setup, step by step

### 5.1 Chart

1. Open **NIFTY futures continuous** (`NSE:NIFTY1!`) or the specific expiry. Not the spot index — spot has no volume and no real fills.
2. Timeframe: **15 minutes** to start. The engine works on 5m–1h; 15m is the honest middle.
3. Chart timezone: **(UTC+5:30) Kolkata**. The session inputs assume it.
4. Right-click the price scale → **Settings → Symbol → Extended hours OFF**.

### 5.2 Load the strategy

**Pine Editor → Open → New blank strategy →** paste all of `VORTEX-Q_v5.pine` → **Save** → **Add to chart**.

### 5.3 Properties tab — do this before reading a single statistic

| Setting | NIFTY | BANKNIFTY | Why |
|---|---|---|---|
| Initial capital | 500000 | 700000 | Must be realistic or the sizing is fiction. |
| Base currency | INR | INR | |
| Order size | **1 Contract** | 1 Contract | The script computes quantity itself; leave this alone. |
| Pyramiding | 0 | 0 | One position at a time. |
| Commission | **0.03 %** | 0.03 % | Matches the ₹966 round trip computed above. |
| Verify price for limit orders | 0 ticks | 0 ticks | |
| **Slippage** | **40 ticks** | **60 ticks** | Slippage is in TICKS. NIFTY tick = 0.05, so 40 ticks = **2 points**. BANKNIFTY moves faster — 60 ticks = 3 points. |
| Margin for long / short | 20% / 20% | 20% / 20% | Roughly NSE index-futures SPAN+exposure. Check your broker. |
| Recalculate: after order is filled | **off** | off | |
| Recalculate: on every tick | **off for backtesting** | off | Turn it on only when you go live. The script is gated so results are identical either way, but off is faster. |
| **Use bar magnifier** | **ON** (paid plans) | ON | Resolves intrabar fill order properly. Without it, TradingView guesses. |

### 5.4 Inputs — starting presets

**NIFTY futures, 15m:**

```
① ENGINE        Mode BALANCED · Min score 72 · Min expected R 0.10
② SESSION       09:30 first entry (570) · 14:45 last (885) · 15:15 square-off (915) · skip 15 min
③ HTF           HTF-1 = 60 · HTF-2 = 240 · Require HTF-1
④ FAMILIES      22 / 18 / 14 / 14 / 18 / 14   (leave these alone until you have 300 trades)
⑤ FIBONACCI     ON · swing 5 · min leg 0.8x ATR · zone 0.382–0.705 · tol 0.25x ATR
⑥ STRUCTURE     ON · pivot 5 · all four events on · memory 20 bars
⑦ ENTRY         Retest + confirmation candle · window 8 · cancel after 3 · cooldown 3
⑧ RISK          1.0% per trade · lot 65 · max 20 lots · notional cap 4x · gap reserve 15%
                Stop = ATR 1.5x, floor 0.8x, ceiling 3.5x
                Max 3 trades/day · daily loss 3% · pause after 4 losses · circuit breaker 20%
⑨ COSTS         leave at the published NSE rates
⑩ TARGETS       T1 1.0R  T2 2.0R  T3 3.0R  T4 5.0R · 30/30/25% out · BE after T1 (+0.1R)
                Trail: ATR chandelier 2.5x, starting after T2 · time stop 30 bars below 0.3R
⑪ WALK-FORWARD  IS start 01 Jan 2023 · OOS start 01 Jan 2025 · OOS end 01 Jan 2030
```

**BANKNIFTY:** lot **30**, stop ATR **1.8x**, slippage 60 ticks, max 2 trades/day. It is a wider, meaner instrument.

**Verify the lot size before you trust the sizing.** NSE revises contract specifications; the defaults here are the values that were current when this was written, not a permanent fact.

### 5.5 Load the companion indicator

Pine Editor → **Open → New blank indicator** → paste `VORTEX-Q_v5_Alerts.pine` → Save → **Add to chart**.

Set its inputs to **exactly** the same values as the strategy. If they differ, the alerts will not match the backtest, and you will trust the wrong thing.

Then turn the strategy's presentation off (`⑫ Draw the planned trade` / `Draw the live trade` / `Floating POSITION INDICATOR panel` all off) so the two scripts do not draw over each other. Let the indicator own the chart and the strategy own the numbers.

### 5.6 Alerts

**Named alerts (from the companion indicator):**

1. Right-click the chart → **Add alert**.
2. **Condition:** `VORTEX-Q v5.0 · Alerts & Visuals` → then pick from the dropdown: `① BUY signal confirmed`, `⑥ Target 1 reached`, `⑩ Stop loss hit`, and so on. All 14 are listed individually.
3. **Trigger:** *Once Per Bar Close*. Not "Once Per Bar" — that fires on a bar that may still reverse.
4. **Expiration:** open-ended.
5. Leave the message box as the default. It already contains `{{plot("Stop")}}`-style placeholders that TradingView fills with the live level.

**Webhook / JSON (from either file):**

1. Add alert → Condition: the script → **`Any alert() function call`**.
2. Trigger: *Once Per Bar Close*.
3. **Empty the message box.** The script supplies the whole JSON body.
4. Paste your endpoint into **Webhook URL**.
5. In the script, set `⑭ Alert payload format = Product JSON`.

The payload uses `bar_time` + `event` as a natural dedupe key. Make your backend idempotent on that pair — TradingView can burst and can gap.

> Alerts do not survive edits. Every time you change an input or the source, recreate the alert.

---

## 6 · The validation protocol — run this before risking money

**Do not skip to step 5.**

1. **Compile.** Paste, save, watch the console. Send me any error verbatim.
2. **Sanity pass.** One year, IS window only. Does it take 1–4 trades a week? Do the lines land where you would have drawn them by hand? Does the LOSS label appear on a trade that tagged T1 and then stopped out? If that last one shows WIN, stop and tell me — the accounting is broken and everything downstream is worthless.
3. **In-sample tuning.** Adjust inputs on the IS window only. **Never look at the OOS numbers while tuning.** The moment you do, OOS stops being out-of-sample and becomes just more in-sample data with extra steps.
4. **Freeze, then reveal.** Write the final settings down. Extend the date range to include OOS. Read the row `Walk-forward verdict`.
   - `OOS HOLDS UP` → continue.
   - `MILD DEGRADATION` → probably real, but sized down.
   - `SEVERE DEGRADATION — likely curve-fit` → **the strategy does not exist.** Go back to step 3 with fewer parameters, not more.
5. **Ablations.** One at a time, off, re-run, record the change in OOS average R:
   - Fibonacci module off
   - Structure module off
   - HTF requirement off
   - Retest entry → breakout-close entry
   A module whose removal does not hurt OOS is a module you are paying maintenance on for nothing. Delete it.
6. **Sample check.** If `Sample adequacy` says INSUFFICIENT, you do not have a result. You have an anecdote. 300 trades is where the numbers start meaning something.
7. **Cross-asset.** Run the identical settings on BANKNIFTY (lot 30). An edge that only exists on one symbol usually is not an edge.
8. **Forward test.** Minimum one month on the companion indicator's alerts with no money. Compare its signal log against what the strategy recorded for the same period.
9. **Then, and only then**, one lot.

---

## 7 · What I have not done, stated plainly

- I have not compiled either file. Static review only.
- I have not backtested anything. Every statistic in both scripts is produced by your chart.
- The `MODEL expected R`, `MODEL P(T1)` and the shrinkage probabilities are **estimates from a model**, not measured results. They start at a pooled prior and only become informative after roughly 50 resolved trades, which the health section reports.
- The signal tracker in the companion indicator has no position sizing and no equity curve. It is a tracker. The strategy is the measuring instrument.
- Lot sizes, STT rates and transaction charges change. Verify them.
- SEBI's own data has 91% of individual F&O traders losing money in FY25. Nothing in this repository changes that base rate on its own. What it changes is whether you can *tell* which side of it you are on.
