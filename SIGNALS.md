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

> The EMA crossover is the **trigger**. RSI, MACD, Volume, and VWAP are the **gates**.

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
RSI acts as a momentum gate on EMA crossovers:
- Crossover fires + RSI > 50 → momentum confirms → **strong signal**
- Crossover fires + RSI ≤ 50 → momentum does not confirm → **weak/grey signal**

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

**In the context of this indicator**
- MFI > 50 is a stronger momentum gate than RSI > 50 alone because it requires volume to confirm the directional move
- MFI overbought (> 80) on a buy signal = caution, entering into potential exhaustion
- Combining MFI + RSI both above 50 gives a high-confidence momentum confirmation

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
- MACD line > signal AND histogram > 0 → **confirms long**
- MACD line < signal AND histogram < 0 → **confirms short**
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

### Using them together

ADX is derived from the DMI calculation — they're inseparable:

1. +DI crosses above −DI → potential long signal
2. −DI crosses above +DI → potential short signal
3. **Only act when ADX > 25** — confirms the trend is real, not noise

A +DI/−DI cross with low ADX is noise. The same cross with ADX > 25 is a meaningful directional signal.

> DMI tells you **which way** the market is moving. ADX tells you **how much to trust it**.

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
- OBV divergence can flag weak signals — price making a new high but OBV isn't confirms the move lacks conviction
- Pairs well with the existing volume dot — volume dot confirms single-bar participation, OBV confirms the cumulative trend of participation over time

**Limitations**
- The absolute value of OBV is meaningless — only direction and divergence from price matter
- One abnormally large volume day can distort the line for many bars
- Does not account for where price closed within the bar — a close up 0.01% adds the full bar's volume, same as a close up 5%

> OBV answers the question: *is volume confirming what price is doing, or quietly telling a different story?*
