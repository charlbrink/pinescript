# Signal Reference — BellsAndWhistles Indicator

A breakdown of every technical component used in the indicator, what it measures, and how it contributes to signal quality.

---

## Table of Contents
1. [EMA — Fast & Slow](#1-ema--fast--slow)
2. [RSI — Momentum Oscillator](#2-rsi--momentum-oscillator)
3. [MFI — Money Flow Index](#3-mfi--money-flow-index)
4. [MACD — Trend Momentum](#4-macd--trend-momentum)
5. [Bollinger Bands — Volatility Context](#5-bollinger-bands--volatility-context)
6. [Volume — Participation Filter](#6-volume--participation-filter)
7. [VWAP — Intraday Bias](#7-vwap--intraday-bias)
8. [Pivot Points — Support & Resistance](#8-pivot-points--support--resistance)
9. [ADX & DMI — Trend Strength & Direction](#9-adx--dmi--trend-strength--direction)
10. [OBV — On Balance Volume](#10-obv--on-balance-volume)
11. [ATR — Stop Loss](#11-atr--stop-loss)
12. [Fibonacci — Golden Pocket](#12-fibonacci--golden-pocket)
13. [Ichimoku Cloud](#13-ichimoku-cloud)
14. [Signal Logic — How All Filters Combine](#14-signal-logic--how-all-filters-combine)

---

## 1. EMA — Fast & Slow

EMA (Exponential Moving Average) weights recent prices more heavily than older ones, making it more responsive than a simple moving average (SMA).

**Why exponential weighting matters**
- SMA treats every bar equally — a close from 20 days ago carries the same weight as today
- EMA prioritises recent bars — yesterday's close matters more than last week's
- Result: EMA tracks price more closely and reacts faster to new information

**Fast vs Slow**

| | Fast EMA (9) | Slow EMA (21) |
|---|---|---|
| Reacts to price | Quickly | Slowly |
| Noise | More | Less |
| Represents | Short-term momentum | Broader trend |

**Crossover signals**
- Fast crosses **above** slow → recent momentum outpacing the trend → bullish shift
- Fast crosses **below** slow → momentum weakening relative to trend → bearish shift
- Gap width = trend strength. Wide = strong trend, narrow = weakening or consolidating

**Common EMA pairs**

| Pair | Use case |
|---|---|
| 9 / 21 | Short-term, intraday and swing (used in this indicator) |
| 12 / 26 | Medium-term (same values MACD is built on) |
| 50 / 200 | Golden cross / Death cross — major trend signals on daily charts |

**Ribbon fill (this indicator)**
- 🟢 Green fill = fast above slow = bullish momentum
- 🔴 Red fill = fast below slow = bearish momentum
- Fill width = visual proxy for trend strength

**Limitations**
- Lags — by the time the cross fires, part of the move is already done
- Whipsaws in choppy/sideways markets — fast and slow keep crossing with no follow-through
- A crossover in a tight range carries little weight — context matters

> The EMA crossover is the **trigger**. RSI, MFI, MACD, OBV, and VWAP are the **gates**.

---

## 2. RSI — Momentum Oscillator

RSI (Relative Strength Index) measures the speed and magnitude of price changes, scaled 0–100.

**Key levels**

| Level | Meaning |
|---|---|
| > 70 | Overbought — price moved fast, may pull back |
| < 30 | Oversold — price sold off hard, may bounce |
| > 50 | Bullish momentum — buyers in control |
| < 50 | Bearish momentum — sellers in control |

**How it's used in this indicator**
RSI acts as one of two momentum gates on EMA crossovers, working alongside MFI:
- Crossover fires + RSI > 50 + MFI > 50 + MACD confirms → **strong signal**
- Either RSI or MFI fails → crossover is treated as **weak/grey**

**Common strategies**
- **Overbought/oversold** — works well in ranging markets, unreliable in strong trends where RSI can stay extreme for extended periods
- **50 crossover** — RSI crossing above 50 confirms bullish momentum, below 50 confirms bearish
- **Divergence** — one of the more reliable RSI signals
  - *Bearish:* price makes higher high, RSI makes lower high → momentum weakening
  - *Bullish:* price makes lower low, RSI makes higher low → selling pressure fading
- **Failure swings** — RSI peaks above 70, pulls back, fails to reclaim 70 on the next push → sell signal based purely on RSI structure

**Limitations**
- Lags — derived from past closes
- Overbought/oversold signals are nearly useless in strong trending markets
- Divergence can persist for many bars before price reacts

---

## 3. MFI — Money Flow Index

MFI is often called the "volume-weighted RSI." It combines both price and volume into a single 0–100 oscillator, making it a stronger momentum filter than RSI alone.

**How it's calculated**

```
Typical price  = (high + low + close) / 3
Money flow     = typical price × volume
Positive flow  = money flow on bars where typical price > previous typical price
Negative flow  = money flow on bars where typical price < previous typical price
MFI            = 100 − (100 / (1 + positive flow / negative flow))  over 14 periods
```

**Key levels**

| Level | Meaning |
|---|---|
| > 80 | Overbought — buying pressure may be exhausted |
| < 20 | Oversold — selling pressure may be exhausted |
| > 50 | Bullish money flow dominant |
| < 50 | Bearish money flow dominant |

**How it differs from RSI**

| | RSI | MFI |
|---|---|---|
| Input | Price only | Price + volume |
| Overbought at | 70 | 80 |
| Oversold at | 30 | 20 |
| Volume weighted | No | Yes |

Because MFI incorporates volume, an overbought reading on high volume carries more weight than the same reading on low volume — RSI cannot make this distinction.

**Common strategies**
- **Overbought/oversold** — MFI > 80 signals buying exhaustion, < 20 signals selling exhaustion. More reliable than RSI in this role because volume confirms the move
- **50 crossover** — MFI crossing above 50 confirms money is flowing in, below 50 confirms outflow. Can replace or sit alongside RSI > 50 as a momentum gate
- **Divergence** — price makes higher high, MFI makes lower high = smart money not participating in the move despite price rising
- **Failure at extremes** — MFI reaches > 80 then fails to reclaim it on the next push = exhaustion signal, similar to RSI failure swings

**How it's used in this indicator**
MFI works alongside RSI as a dual momentum gate — both must confirm for a strong signal:
- Crossover fires + MFI > 50 + RSI > 50 + MACD confirms → **strong signal**
- MFI ≤ 50 → crossover demoted to **weak/grey**
- MFI > 80 on a buy crossover → demoted to **CAUTION ▲** (grey label) — valid momentum but entering into potential exhaustion
- MFI < 50 + RSI < 50 + MACD confirms → **strong short signal**

**Limitations**
- Lags — based on a lookback period of past bars
- In low-volume environments (pre-market, thin instruments) the volume weighting can produce misleading readings
- Like RSI, can stay overbought/oversold for extended periods in strong trends

---

## 4. MACD — Trend Momentum

MACD (Moving Average Convergence Divergence) is a trend-following momentum indicator built from three components:

| Component | Calculation |
|---|---|
| MACD line | 12 EMA − 26 EMA |
| Signal line | 9 EMA of the MACD line |
| Histogram | MACD line − Signal line |

**Reading the components**
- MACD line crosses **above** signal line → bullish momentum building
- MACD line crosses **below** signal line → bearish momentum building
- Histogram **above zero** → MACD above signal, positive momentum
- Histogram **shrinking toward zero** → momentum fading, crossover may be coming

**How it's used in this indicator**
Both the signal line cross AND histogram direction must agree:
- MACD line > signal AND histogram > 0 → **confirms long** (combined with RSI > 50 and MFI > 50)
- MACD line < signal AND histogram < 0 → **confirms short** (combined with RSI < 50 and MFI < 50)
- Either condition fails → crossover is treated as **weak**

**Common strategies**
- **Signal line crossover** — most basic use, noisy in choppy markets
- **Zero line cross** — MACD crosses above zero = 12 EMA now above 26 EMA, trend has turned bullish. Slower but more reliable
- **Histogram direction** — histogram growing = momentum accelerating, shrinking = weakening even if price is still moving. Useful as an early warning before the line cross
- **Divergence** — often more reliable than RSI divergence
  - *Bearish:* price makes higher high, histogram makes lower high
  - *Bullish:* price makes lower low, histogram makes higher low

**Reading histogram shifts early**
The histogram is often more useful than the crossover itself. When it starts shrinking before the lines actually cross, that's the early warning that momentum is shifting — traders watching the histogram can position ahead of the signal line cross.

**Limitations**
- Lags significantly — built on EMAs of EMAs
- Whipsaws in sideways markets
- A signal line cross near the zero line is much weaker than one far from zero — distance from zero matters
- Does not give price targets

---

## 5. Bollinger Bands — Volatility Context

Bollinger Bands are a volatility envelope — three lines plotted around price.

| Band | Calculation |
|---|---|
| Upper | 20 SMA + 2 standard deviations |
| Middle | 20 SMA (baseline) |
| Lower | 20 SMA − 2 standard deviations |

**Core concepts**
- Price tends to revert to the middle band — it acts as a magnet
- ~95% of price action stays inside the bands under normal conditions
- Bands **widen** during high volatility, **contract** during low volatility (squeeze)

**How it's used in this indicator**
Plotted as a visual reference only — not used to gate signals. The bands show whether price is extended (near the outer bands) or compressed (near the midline) relative to where an EMA crossover fires.

**Visual style**
- Outer bands (upper/lower) — silver, linewidth 1
- Middle band — yellow, linewidth 1

**Common strategies**
- **Mean reversion** — price touches upper band → potential short, lower band → potential long. Works in ranging markets, dangerous in strong trends
- **Squeeze breakout** — bands tighten significantly → a large move is building. Direction unknown until price breaks out; combine with volume to confirm
- **Trend riding** — in a strong uptrend, price walks the upper band. Pullbacks to the midline are buy opportunities, not shorts
- **Band rejection** — price closes outside the band then closes back inside = exhaustion signal, often precedes a reversal

**Limitations**
- Does not give price targets — price can walk a band for many bars
- Squeeze alone is not a signal — a catalyst or confirmation is needed
- Mean reversion trades against a strong trend will get chopped up

---

## 6. Volume — Participation Filter

Volume confirms whether real money is behind a move.

**How it's used in this indicator**
- Volume MA (20-period SMA) is calculated
- Current volume > volume MA → participation confirmed → dot plotted at the bottom of the chart
- A signal firing on below-average volume is suspect regardless of other conditions

**Why it matters**
- Price moves on low volume can be easily reversed — there's no conviction behind them
- High volume on a breakout or crossover means institutions and large players are participating
- Low volume during a rally = distribution (smart money selling into retail buyers)

**Key combinations**
- EMA crossover + volume spike = high conviction signal
- MACD crossover + rising volume = momentum is real
- BB breakout + volume surge = squeeze resolution with participation

---

## 7. VWAP — Intraday Bias

VWAP (Volume Weighted Average Price) is the average price paid for every share traded that session, weighted by volume. It resets each day.

**What it represents**
VWAP is the benchmark institutional traders use to evaluate their own execution quality. It acts as a fair value line for the session:

| Price location | Implication |
|---|---|
| Above VWAP | Buyers in control all day — institutions net long |
| Below VWAP | Sellers in control all day — institutions net short |

**How it's used in this indicator**
Acts as a bias gate — the VWAP dot at the top of the chart shows current bias:
- 🟢 Green dot = price above VWAP = long bias
- 🔴 Red dot = price below VWAP = short bias

**Why it works**
Price above VWAP means institutions have been buying all day and are sitting on unrealised gains — they're unlikely to flip short aggressively. Below VWAP is the opposite. Trading in the direction of VWAP bias means trading with the dominant institutional flow.

**Limitations**
- Intraday tool only — resets each session, loses meaning on daily/weekly charts
- Most powerful on 1m–1H timeframes where session context is relevant
- Automatically disabled on daily and higher timeframes in this indicator

---

## 8. Pivot Points — Support & Resistance

Pivot points are price levels calculated from the previous session's high, low, and close. A classic tool used by floor traders and institutions to identify where price is likely to find support or resistance.

**Calculation**

```
PP  = (prev high + prev low + prev close) / 3
R1  = (2 × PP) − prev low
R2  = PP + (prev high − prev low)
S1  = (2 × PP) − prev high
S2  = PP − (prev high − prev low)
```

**The five levels**

| Level | Role |
|---|---|
| PP | Central pivot — price above = bullish bias, below = bearish bias |
| R1, R2 | Resistance — price likely to stall or reverse |
| S1, S2 | Support — price likely to bounce |

**Signal quality near pivots**
- BUY signal at S1 or S2 = stronger — price bounced off known support AND indicators aligned
- SELL signal at R1 or R2 = stronger — price rejected known resistance AND indicators confirmed
- BUY signal running into R1 above = risky — buying straight into a ceiling
- SELL signal running into S1 below = risky — shorting straight into a floor

Pivot levels add a **location quality** layer — not just *is the trend right* but *is this a good price to enter*.

---

## 9. ADX & DMI — Trend Strength & Direction

ADX and DMI are derived from the same calculation and are always used together.

### ADX — Average Directional Index

Measures **trend strength** — not direction, only how strong the trend is.

| ADX Value | Market condition |
|---|---|
| < 20 | Weak or no trend — choppy, avoid trend-following signals |
| 20–25 | Trend developing |
| > 25 | Trending market — signals carry more weight |
| > 40 | Strong trend |

### DMI — Directional Movement Index

Two lines that show **trend direction**:
- **+DI** — measures upward price movement pressure
- **−DI** — measures downward price movement pressure

| Condition | Implication |
|---|---|
| +DI > −DI | Bulls in control |
| −DI > +DI | Bears in control |

**How it's used in this indicator**
Three visual layers, all configurable via indicator settings:

- **ADX/DMI Bias dot** (toggle: Show ADX/DMI Dots) — plotted above every bar
  - 🟢 Green = ADX > 25 and +DI > −DI (trending bullish)
  - 🔴 Red = ADX > 25 and −DI > +DI (trending bearish)
  - ⚪ Grey = ADX < 25 (no trend / choppy)
- **DMI Crossover markers** (toggle: Show DMI Crossovers) — xcross shape plotted when +DI crosses −DI, only when ADX > 25
  - 🟢 Green xcross below bar = +DI crossed above −DI
  - 🔴 Red xcross above bar = −DI crossed above +DI
- **Live ADX label** — displayed at the right edge of the chart, colour coded
  - Green = ADX > 25 (trending)
  - Yellow = ADX 20–25 (developing)
  - Red = ADX < 20 (choppy)

---

## 10. OBV — On Balance Volume

OBV is a cumulative running total of volume, based on the direction of each close:
- Close **up** from previous bar → volume is **added**
- Close **down** from previous bar → volume is **subtracted**

The result is a single line that rises when buying volume dominates and falls when selling volume dominates.

**The core idea**
Volume precedes price. Smart money accumulates or distributes before price moves — OBV picks this up before the price chart does. A rising OBV means buyers are absorbing supply even if price hasn't moved yet.

**How it's used**

| Signal | Meaning |
|---|---|
| Price rising + OBV rising | Healthy trend — real buying behind the move |
| Price rising + OBV flat/falling | Distribution — move is suspect, smart money selling into strength |
| Price falling + OBV falling | Healthy downtrend — real selling pressure |
| Price falling + OBV flat/rising | Accumulation — sellers exhausted, potential reversal |

**Common strategies**
- **Trend confirmation** — OBV should trend in the same direction as price. When they diverge, the price move is likely unsustainable
- **Divergence** — most powerful use of OBV
  - *Bearish:* price makes new high, OBV fails to make new high → buyers fading, distribution underway
  - *Bullish:* price makes new low, OBV fails to make new low → sellers exhausted, accumulation underway
- **Breakout confirmation** — OBV breaking above a prior high before price does = early signal a price breakout is coming

**In the context of this indicator**
OBV bearish divergence is used to demote long signals:
- Price reaches a 20-bar high but OBV is below its prior 20-bar high → bearish divergence detected
- Any `longCondition` that fires during OBV bearish divergence is demoted to **WEAK ▲** (grey)
- Pairs well with the volume dot — the dot confirms single-bar participation, OBV confirms the cumulative trend of participation over time

**Limitations**
- The absolute value of OBV is meaningless — only direction and divergence from price matter
- One abnormally large volume day can distort the line for many bars
- Does not account for where price closed within the bar — a close up 0.01% adds the full bar's volume, same as a close up 5%

> OBV answers the question: *is volume confirming what price is doing, or quietly telling a different story?*

---

## 11. ATR — Stop Loss

ATR (Average True Range) measures the average price range per bar over a lookback period. On the daily timeframe it gives a volatility-adjusted distance to place a stop that accounts for normal daily price movement.

**Calculation**
```
True Range  = max(high − low, abs(high − prev close), abs(low − prev close))
ATR(14)     = 14-period smoothed average of True Range
Stop dist.  = ATR(14) × multiplier
```

**How it's used in this indicator**
- ATR is pulled from the **daily timeframe** regardless of the chart timeframe — gives a consistent stop distance based on daily volatility
- On a BUY signal: stop level = `close − (ATR × multiplier)`
- On a SELL signal: stop level = `close + (ATR × multiplier)`
- A solid red line (long SL) or green line (short SL) is drawn 5 bars forward from the signal bar with an "SL" label at the left end
- The stop price is also shown in the BUY/SELL label: `BUY ▲ 189.45  SL: 187.20`

**Configurable input**
- `ATR Stop Multiplier` (default 2.0, step 0.5) — tune per instrument and timeframe
  - 1.5 = tight stop, suits lower volatility instruments
  - 2.0 = standard
  - 3.0 = wide/swing stop, suits higher volatility instruments

**Limitations**
- ATR is a distance, not a direction — it doesn't tell you if the stop is at a logical price level (e.g. below support)
- In low volatility periods ATR shrinks, placing stops very tight — can get stopped out on normal noise
- In high volatility periods ATR expands, placing stops far away — increases risk per trade

---

## 12. Fibonacci — Golden Pocket

Fibonacci retracement levels are horizontal zones derived from the Fibonacci sequence, used to identify where price is likely to find support or resistance during a pullback.

**The Golden Pocket (0.382–0.618)**

The 0.382–0.618 zone is the most consistently traded retracement range across timeframes and instruments. It represents the area where price most commonly pauses or reverses during a healthy pullback before continuing in the trend direction.

| Level | Meaning |
|---|---|
| 0.382 | Shallow retracement — trend is strong, buyers stepping in early |
| 0.5 | Mid-point retracement — common pause zone |
| 0.618 | Deep retracement — last line of defence before trend is considered broken |

**How it's used in this indicator**
- Pivot highs and lows are auto-detected using a 21-bar lookback on each side via `ta.pivothigh` / `ta.pivotlow`
- The most recent pivot high and low define the swing range
- 0.382 and 0.618 levels are calculated from that range and plotted as yellow lines
- The zone between them is filled in yellow — this is the golden pocket
- Toggle via "Show Fibonacci Golden Pocket" in settings

**Note on lag**
Because pivot detection requires 21 bars on each side to confirm, levels update 21 bars after a new swing forms. This is the tradeoff for using a significant lookback — fewer false swings, but slightly delayed.

**Signal quality near the golden pocket**
- BUY signal firing inside the golden pocket during an uptrend = high-conviction entry — price pulled back into the expected retracement zone and indicators confirmed
- SELL signal firing inside the golden pocket during a downtrend = same logic in reverse
- A signal firing well outside the zone carries more risk

---

## 13. Ichimoku Cloud

The Ichimoku Cloud is a trend-following system with 5 components.

### The Lines

| Line | Color | Description |
|---|---|---|
| Tenkan-sen | Fuchsia | 9-period midpoint — the "fast" line. Acts like a short-term moving average. Price crossing it signals short-term momentum shifts |
| Kijun-sen | Orange | 26-period midpoint — the "slow" line. Acts as dynamic support/resistance. A Tenkan/Kijun crossover is a primary trade signal |
| Chikou Span | Lime green | Today's close plotted 26 bars back. If it's above the price from 26 bars ago = bullish. Below = bearish |
| Span A | Cyan (`#00e5ff`) | Plotted 26 bars ahead — upper boundary of bullish cloud |
| Span B | Coral (`#FF7F50`) | Plotted 26 bars ahead — lower boundary of bullish cloud |

### The Cloud (Kumo)

- Span A and Span B are plotted 26 bars into the future
- The space between them is the cloud
- Cyan cloud = bullish, coral cloud = bearish
- Thick cloud = strong support/resistance. Thin cloud = weak, easy to break through

### How to Read Signals

| Condition | Meaning |
|---|---|
| Price above cloud | Uptrend |
| Price below cloud | Downtrend |
| Price inside cloud | Choppy / neutral |
| Tenkan crosses above Kijun while price is above cloud | Strong buy |
| Tenkan crosses below Kijun while price is below cloud | Strong sell |
| Crossover inside the cloud | Weak / unreliable signal |
| Chikou above price from 26 bars ago | Bullish momentum confirmed |

### Confluence with This Indicator

The strongest setups occur when all of the following align:
- Green BUY triangle fires
- Price is above the cyan cloud
- Chikou is above its reference price
- ADX dot is green (trending bullish)

All 4 aligning is a high-conviction entry.

---

## 14. Signal Logic — How All Filters Combine

Every signal on the chart is the result of multiple filters agreeing. This section documents exactly what conditions produce each signal type.

**Strong BUY ▲ (green triangle + label)**

All of the following must be true:
- EMA fast crosses above EMA slow (`crossUp`)
- RSI(14) > 50
- MFI(14) > 50
- MFI(14) ≤ 80 (not overbought)
- MACD line > signal line AND histogram > 0
- No OBV bearish divergence (price not at 20-bar high while OBV is below its prior high)

**Strong SELL ▼ (red triangle + label)**

All of the following must be true:
- EMA fast crosses below EMA slow (`crossDown`)
- RSI(14) < 50
- MFI(14) < 50
- MACD line < signal line AND histogram < 0

**WEAK ▲ / WEAK ▼ (grey triangle + label)**

A crossover fired but one or more filters failed:
- RSI, MFI, or MACD did not confirm direction
- OBV bearish divergence detected on a long crossover

**CAUTION ▲ (grey triangle + label)**

A long crossover fired with RSI/MACD confirming but MFI > 80 — momentum is present but buying pressure may be exhausted.

**Visual indicator dots**

| Dot | Location | Meaning |
|---|---|---|
| VWAP bias circle | Top of chart | Green = above VWAP, Red = below VWAP, Grey = daily+ timeframe |
| MFI diamond (toggleable) | Below candle | Green = MFI > 50, Orange = MFI > 80, Red = MFI < 50 |
| ADX/DMI circle (toggleable) | Above candle | Green = trending bullish, Red = trending bearish, Grey = choppy |
| Volume circle | Bottom of chart | Teal = current volume above 20-bar average |
| DMI xcross (toggleable) | Above/below candle | Green = +DI crossed above −DI, Red = −DI crossed above +DI (ADX > 25 only) |
