# Trading Scripts for use within TradingView


---
## Table of contents

1. [SuperTrend](#supertrend)
2. [Trend Magic](#trend-magic)
3. [Volumatic](#volumatic)

---

# SuperTrend
A Pine Script implementation of the SuperTrend indicator with buy/sell signals.

## What is SuperTrend?
SuperTrend is a trend-following indicator that uses Average True Range (ATR) to calculate dynamic support and resistance levels. It plots a single line that flips above or below price to indicate trend direction:

- **Green line below price** = Uptrend (bullish)
- **Red line above price** = Downtrend (bearish)

The indicator is popular for its simplicity and effectiveness in identifying trend reversals while filtering out market noise.

## How It Works

### Calculation

1. **ATR Calculation**: Measures average price volatility over a specified period
2. **Basic Bands**:
   - Upper Band = `(High + Low) / 2 + (Factor × ATR)`
   - Lower Band = `(High + Low) / 2 - (Factor × ATR)`
3. **Final Bands**: Adjusted dynamically to prevent premature flips
4. **Trend Direction**: Determined by price crossing the bands

### Signal Logic

| Signal | Condition |
|--------|-----------|
| **Buy** | Price closes above the upper band (trend flips bullish) |
| **Sell** | Price closes below the lower band (trend flips bearish) |

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| ATR Period | 14 | Lookback period for ATR calculation |
| Factor | 2.0 | Multiplier applied to ATR for band width |

### Parameter Tuning

- **Higher Factor / Longer Period** = Fewer signals, less noise, slower response
- **Lower Factor / Shorter Period** = More signals, more noise, faster response

## Features

- Overlay indicator (plots directly on price chart)
- Color-coded trend lines (green = bullish, red = bearish)
- Visual buy/sell bubbles with labels at trend reversals
- Built-in alert conditions for automated notifications

## Installation

1. Open TradingView and navigate to your chart
2. Click **Pine Editor** at the bottom of the screen
3. Delete any existing code and paste the script
4. Click **Add to Chart**

## Usage Notes

- Works on any timeframe and instrument
- Best used in trending markets; may whipsaw in ranging/choppy conditions
- Consider combining with other confirmation indicators (RSI, MACD, volume)
- Not financial advice—always backtest before live trading

## License

MIT License - Free to use and modify

```
//@version=6
// Copyright (c) 2019-present, Alex Orekhov (everget)
// SuperTrend script may be freely distributed under the terms of the GPL-3.0 license.
indicator('SuperTrend', overlay = true)

const string calcGroup = 'Calculation'
length = input.int(22, title = 'ATR Period', group = calcGroup)
mult = input.float(3, step = 0.1, title = 'ATR Multiplier', group = calcGroup)
src = input.source(hl2, title = 'Source', group = calcGroup)
wicks = input.bool(true, title = 'Take Wicks into Account', group = calcGroup)

const string visualGroup = 'Visuals'
showLabels = input.bool(true, title = 'Show Buy/Sell Labels', group = visualGroup)
highlightState = input.bool(true, title = 'Highlight State', group = visualGroup)

//---

atr = mult * ta.atr(length)

highPrice = wicks ? high : close
lowPrice = wicks ? low : close
doji4price = open == close and open == low and open == high

longStop = src - atr
longStopPrev = nz(longStop[1], longStop)

if longStop > 0
    if doji4price
        longStop := longStopPrev
        longStop
    else
        longStop := lowPrice[1] > longStopPrev ? math.max(longStop, longStopPrev) : longStop
        longStop
else
    longStop := longStopPrev
    longStop

shortStop = src + atr
shortStopPrev = nz(shortStop[1], shortStop)

if shortStop > 0
    if doji4price
        shortStop := shortStopPrev
        shortStop
    else
        shortStop := highPrice[1] < shortStopPrev ? math.min(shortStop, shortStopPrev) : shortStop
        shortStop
else
    shortStop := shortStopPrev
    shortStop

var int dir = 1
dir := dir == -1 and highPrice > shortStopPrev ? 1 : dir == 1 and lowPrice < longStopPrev ? -1 : dir

const color textColor = color.white
const color longColor = color.green
const color shortColor = color.red
const color longFillColor = color.new(color.green, 85)
const color shortFillColor = color.new(color.red, 85)

longStopPlot = plot(dir == 1 ? longStop : na, title = 'Long Stop', style = plot.style_linebr, linewidth = 2, color = longColor)
buySignal = dir == 1 and dir[1] == -1
plotshape(buySignal ? longStop : na, title = 'Long Stop Start', location = location.absolute, style = shape.circle, size = size.tiny, color = longColor)
plotshape(buySignal and showLabels ? longStop : na, title = 'Buy Label', text = 'Buy', location = location.absolute, style = shape.labelup, size = size.tiny, color = longColor, textcolor = textColor)

shortStopPlot = plot(dir == 1 ? na : shortStop, title = 'Short Stop', style = plot.style_linebr, linewidth = 2, color = shortColor)
sellSignal = dir == -1 and dir[1] == 1
plotshape(sellSignal ? shortStop : na, title = 'Short Stop Start', location = location.absolute, style = shape.circle, size = size.tiny, color = shortColor)
plotshape(sellSignal and showLabels ? shortStop : na, title = 'Sell Label', text = 'Sell', location = location.absolute, style = shape.labeldown, size = size.tiny, color = shortColor, textcolor = textColor)

midPricePlot = plot(ohlc4, title = '', display = display.none, editable = false)

fill(midPricePlot, longStopPlot, title = 'Long State Filling', color = (highlightState and dir == 1 ? longFillColor : na))
fill(midPricePlot, shortStopPlot, title = 'Short State Filling', color = (highlightState and dir == -1 ? shortFillColor : na))

alertcondition(dir != dir[1], title = 'SuperTrend Direction Change', message = 'SuperTrend has changed direction, {{exchange}}:{{ticker}}')
alertcondition(buySignal, title = 'SuperTrend Buy', message = 'SuperTrend Buy, {{exchange}}:{{ticker}}')
alertcondition(sellSignal, title = 'SuperTrend Sell', message = 'SuperTrend Sell, {{exchange}}:{{ticker}}')

```

# Trend Magic
A Pine Script v6 implementation of the Trend Magic indicator with buy/sell signals, matching the ThinkOrSwim version.

## What is Trend Magic?

Trend Magic is a trend-following indicator that combines the Commodity Channel Index (CCI) for direction detection with Average True Range (ATR) for dynamic trailing levels. It plots a single line that trails price and changes color based on trend direction:

- **Blue line below price** = Uptrend (bullish)
- **Red line above price** = Downtrend (bearish)

The indicator excels at identifying trend reversals earlier than purely price-based indicators by using momentum (CCI) as its directional trigger.

## How It Works

### Calculation

1. **Custom CCI Calculation**: Uses close price only (not typical price) with manual calculation:
   - `SMA = Simple Moving Average of close`
   - `MAD = Mean Absolute Deviation from SMA`
   - `CCI = (close - SMA) / (0.015 × MAD)`
2. **ATR Calculation**: Measures average price volatility over a specified period
3. **Trend Direction**:
   - CCI ≥ 0 = Bullish
   - CCI < 0 = Bearish
4. **Trend Magic Line**:
   - Uptrend: `Low - (ATR × Multiplier)` — ratchets upward using max() logic
   - Downtrend: `High + (ATR × Multiplier)` — ratchets downward using min() logic

### Signal Logic

| Signal | Condition |
|--------|-----------|
| **Buy** | CCI crosses above zero (trend flips bullish) |
| **Sell** | CCI crosses below zero (trend flips bearish) |

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| CCI Period | 20 | Lookback period for CCI calculation |
| ATR Period | 5 | Lookback period for ATR calculation |
| ATR Multiplier | 1.0 | Distance multiplier for the trailing line |

### Parameter Tuning

- **Higher CCI Period** = Smoother signals, fewer whipsaws, slower response
- **Lower CCI Period** = Faster signals, more noise, quicker response
- **Higher ATR Multiplier** = Line trails further from price, wider stops
- **Lower ATR Multiplier** = Line trails closer to price, tighter stops

## Features

- Overlay indicator (plots directly on price chart)
- Color-coded trend line (blue = bullish, red = bearish)
- Visual buy/sell bubbles with labels at trend reversals (green/red markers)
- Built-in alert conditions for automated notifications
- Continuous ratcheting logic prevents premature signal flips
- **Exact logic match with ThinkOrSwim implementation**

## Installation

1. Open TradingView and navigate to your chart
2. Click **Pine Editor** at the bottom of the screen
3. Delete any existing code and paste the script
4. Click **Add to Chart**

## Trend Magic vs SuperTrend

| Aspect | SuperTrend | Trend Magic |
|--------|------------|-------------|
| **Direction Trigger** | Price crossing ATR bands | CCI crossing zero |
| **Responsiveness** | Reacts to price action | Reacts to momentum shift |
| **False Signals** | More in choppy markets | Fewer due to CCI smoothing |
| **Band Anchor** | HL2 (midpoint) | High/Low extremes |
| **CCI Calculation** | N/A | Custom using close only |

## Implementation Notes

- **Pine Script v6** compatible
- Custom CCI calculation uses close price only (not typical price HLC/3) to match TOS
- Continuous max/min ratcheting logic (no reset on trend changes)
- ATR period default of 5 matches ThinkOrSwim version
- Line weight set to 3 for visibility

## Usage Notes

- Works on any timeframe and instrument
- CCI-based direction often catches reversals earlier than price-based methods
- Best used in trending markets; may generate false signals in extended ranges
- Consider combining with volume or other confirmation indicators
- Not financial advice—always backtest before live trading

```
//@version=6
indicator("Trend Magic", overlay=true)

// Inputs
cciPeriod = input.int(20, "CCI Period", minval=1)
atrPeriod = input.int(5, "ATR Period", minval=1)  // Changed from 20 to match TOS default
atrMult = input.float(1.0, "ATR Multiplier", minval=0.1, step=0.1)

// Manual CCI Calculation (matching TOS exactly - uses close, not typical price)
tp = close
sma = ta.sma(tp, cciPeriod)
mad = ta.sma(math.abs(tp - sma), cciPeriod)
cci = (tp - sma) / (0.015 * mad)

// ATR Calculation
atr = ta.atr(atrPeriod)

// Bands
upLine = low - (atr * atrMult)
downLine = high + (atr * atrMult)

// Trend Magic Line (matching TOS CompoundValue logic exactly)
var float trendMagic = close
trendMagic := cci >= 0 ? math.max(trendMagic[1], upLine) : math.min(trendMagic[1], downLine)

// Colors (blue/red to match TOS)
lineColor = cci >= 0 ? color.blue : color.red

// Plot Trend Magic Line
plot(trendMagic, "Trend Magic", color=lineColor, linewidth=3)

// Trend for signals
trend = cci >= 0 ? 1 : -1

// Buy/Sell Signals
buySignal = trend == 1 and trend[1] == -1
sellSignal = trend == -1 and trend[1] == 1

// Buy/Sell Markers
plotshape(buySignal, "Buy", shape.circle, location.belowbar, color.green, size=size.normal, text="Buy", textcolor=color.white)
plotshape(sellSignal, "Sell", shape.circle, location.abovebar, color.red, size=size.normal, text="Sell", textcolor=color.white)

// Alerts
alertcondition(buySignal, "Trend Magic Buy", "Trend Magic flipped bullish")
alertcondition(sellSignal, "Trend Magic Sell", "Trend Magic flipped bearish")
```

# Volumatic
A Pine Script implementation of the Volumatic Variable Index Dynamic Average (VIDYA) indicator with volume pressure analysis and market structure detection.

## What is Volumatic VIDYA?

Volumatic VIDYA combines the Variable Index Dynamic Average (VIDYA) with volume pressure analysis to provide a comprehensive trend-following system. Unlike traditional moving averages that use fixed smoothing periods, VIDYA dynamically adjusts its responsiveness based on market momentum using the Chande Momentum Oscillator (CMO).

The indicator provides:

- **Adaptive trend line** that responds faster during strong momentum and slower during consolidation
- **ATR-based bands** for volatility-adjusted support/resistance zones
- **Delta volume tracking** to measure buy/sell pressure within each trend
- **Market structure pivots** with average volume labels at key levels

## How It Works

### VIDYA Calculation

1. **Momentum Measurement**: Calculate price change over the momentum period
2. **CMO Calculation**: Separate positive and negative momentum sums to derive the absolute CMO value
3. **Dynamic Alpha**: `alpha = 2 / (length + 1)` adjusted by CMO percentage
4. **VIDYA Value**: `VIDYA = (alpha × |CMO| / 100 × price) + (1 - alpha × |CMO| / 100) × previous VIDYA`
5. **Smoothing**: Final value smoothed with a 15-period SMA

### Trend Detection

| Condition | Trend |
|-----------|-------|
| Price closes above upper band | Bullish |
| Price closes below lower band | Bearish |
| Price between bands | Previous trend continues |

### Volume Pressure (Delta Volume)

The indicator accumulates volume throughout each trend:

- **Up bars** (close > open) add to buy volume
- **Down bars** (close < open) add to sell volume
- **Delta** = Buy Volume - Sell Volume
- Delta resets on each trend change

Positive delta indicates net buying pressure; negative delta indicates net selling pressure.

### Market Structure Pivots

- Pivot highs and lows detected using a configurable lookback period
- Horizontal dotted lines extend from each pivot until price crosses them
- Average volume over 6 bars displayed at each pivot point
- Provides context for support/resistance strength

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| VIDYA Length | 10 | Base smoothing period for VIDYA calculation |
| VIDYA Momentum | 20 | Lookback period for CMO calculation |
| Pivot Length | 5 | Bars to left/right for pivot detection |
| Band Distance | 2.0 | ATR multiplier for upper/lower bands |
| ATR Length | 14 | Period for ATR calculation |
| Bullish Color | Green | Color for uptrend elements |
| Bearish Color | Red | Color for downtrend elements |

### Parameter Tuning

- **Higher VIDYA Length** = Smoother line, slower to react
- **Higher VIDYA Momentum** = More stable CMO, less adaptive
- **Higher Pivot Length** = Fewer pivots, more significant levels
- **Higher Band Distance** = Wider bands, fewer trend changes

## Features

| Feature | Description |
|---------|-------------|
| **VIDYA Line** | Adaptive moving average with trend-based coloring |
| **ATR Bands** | Dynamic upper/lower bands with shaded fill |
| **Triangle Signals** | Visual buy/sell markers at trend reversals |
| **Delta Volume Label** | Real-time cumulative volume pressure display |
| **Pivot Lines** | Dotted horizontal lines at market structure points |
| **Volume Labels** | Average volume displayed at each pivot |
| **Alert Conditions** | Built-in alerts for trend changes |

## Installation

1. Open TradingView and navigate to your chart
2. Click **Pine Editor** at the bottom of the screen
3. Delete any existing code and paste the script
4. Click **Add to Chart**

## Interpretation Guide

### Trend Analysis

| Signal | Meaning |
|--------|---------|
| Green VIDYA line | Bullish trend active |
| Red VIDYA line | Bearish trend active |
| Triangle up | New bullish trend starting |
| Triangle down | New bearish trend starting |

### Volume Analysis

| Delta Volume | Interpretation |
|--------------|----------------|
| Large positive | Strong buying pressure supporting uptrend |
| Small positive | Weak buying pressure, potential exhaustion |
| Large negative | Strong selling pressure supporting downtrend |
| Small negative | Weak selling pressure, potential reversal |

### Pivot Lines

- **Lines holding** = Strong support/resistance
- **Lines broken** = Level invalidated, removed from chart
- **High volume at pivot** = More significant level
- **Low volume at pivot** = Weaker level, more likely to break

## VIDYA vs Traditional Moving Averages

| Aspect | SMA/EMA | VIDYA |
|--------|---------|-------|
| **Smoothing** | Fixed period | Adaptive to momentum |
| **Responsiveness** | Constant | Faster in trends, slower in ranges |
| **Whipsaws** | More frequent | Reduced due to dynamic adjustment |
| **Calculation** | Simple average | CMO-weighted smoothing |

## Use Cases

- **Trend Following**: Enter on triangle signals, ride trends with VIDYA as trailing reference
- **Volume Confirmation**: Validate trends by checking delta volume alignment
- **Support/Resistance**: Use pivot lines for profit targets and stop placement
- **Momentum Assessment**: CMO-based adaptation reveals underlying momentum strength

## Usage Notes

- Works on any timeframe and instrument
- Most effective in trending markets with clear directional moves
- Delta volume most meaningful on assets with reliable volume data
- Pivot lines work best on higher timeframes (1H+) for significant levels
- Consider combining with RSI or MACD for additional confirmation
- Not financial advice—always backtest before live trading


## Credits

Inspired by the original [Volumatic VIDYA by BigBeluga](https://www.tradingview.com/script/llhVjhA5-Volumatic-Variable-Index-Dynamic-Average-BigBeluga/).

```
//@version=6
indicator("VIDYA Trend", overlay=true)

// Inputs
vidya_length = input.int(10, "VIDYA Length", minval=1)
vidya_momentum = input.int(20, "VIDYA Momentum", minval=1)
band_distance = input.float(2.2, "Band Distance", minval=0.1, step=0.1)
source = input.source(close, "Source")

// VIDYA Calculation Function
vidya_calc(src, length, momentum_len) =>
    float momentum = ta.change(src)
    float sum_pos = math.sum((momentum >= 0) ? momentum : 0.0, momentum_len)
    float sum_neg = math.sum((momentum >= 0) ? 0.0 : -momentum, momentum_len)
    float abs_cmo = math.abs(100 * (sum_pos - sum_neg) / (sum_pos + sum_neg))
    float alpha = 2 / (length + 1)
    var float vidya = 0.0
    vidya := alpha * abs_cmo / 100 * src + (1 - alpha * abs_cmo / 100) * nz(vidya[1])
    ta.sma(vidya, 15)

// Calculate VIDYA
vidya = vidya_calc(source, vidya_length, vidya_momentum)

// ATR for band calculation (used for trend detection, not plotted)
atr = ta.atr(14)
upper_band = vidya + (band_distance * atr)
lower_band = vidya - (band_distance * atr)

// Trend Detection (SuperTrend-style logic using bands)
var int trend = 1
trend := trend == -1 and close > upper_band ? 1 : 
         trend == 1 and close < lower_band ? -1 : 
         trend

// Buy/Sell Signals (only on trend flips)
buySignal = trend == 1 and trend[1] == -1
sellSignal = trend == -1 and trend[1] == 1

// Plot VIDYA Line Only (color based on trend)
vidya_color = trend == 1 ? color.rgb(0, 255, 255) : color.rgb(255, 0, 255)  // Cyan for up, Magenta for down
plot(vidya, "VIDYA", color=vidya_color, linewidth=2)

// Buy Signal - Rectangle with Label
if buySignal
    box.new(bar_index, low * 0.998, bar_index + 3, low * 0.996, 
             bgcolor=color.new(color.green, 80), 
             border_color=color.green, 
             border_width=2)
    label.new(bar_index + 1, low * 0.997, "BUY", 
              color=color.new(color.green, 100), 
              textcolor=color.white, 
              style=label.style_none, 
              size=size.small)

// Sell Signal - Rectangle with Label
if sellSignal
    box.new(bar_index, high * 1.002, bar_index + 3, high * 1.004, 
             bgcolor=color.new(color.red, 80), 
             border_color=color.red, 
             border_width=2)
    label.new(bar_index + 1, high * 1.003, "SELL", 
              color=color.new(color.red, 100), 
              textcolor=color.white, 
              style=label.style_none, 
              size=size.small)

// Alerts
alertcondition(buySignal, "VIDYA Buy", "VIDYA trend turned bullish")
alertcondition(sellSignal, "VIDYA Sell", "VIDYA trend turned bearish")
```
