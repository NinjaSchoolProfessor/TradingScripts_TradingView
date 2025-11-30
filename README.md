# Trading Scripts for use within TradingView


---
## Table of contents

1. [SuperTrend](#supertrend)
2. [Trend Magic](#trend-magic)

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
//@version=5
indicator("Super Trend", overlay=true)

// Inputs
atrPeriod = input.int(14, "ATR Period", minval=1)
factor = input.float(2.0, "Factor", minval=0.1, step=0.1)

// ATR Calculation
atr = ta.atr(atrPeriod)

// Basic Bands
hl2_val = hl2
upperBasic = hl2_val + (factor * atr)
lowerBasic = hl2_val - (factor * atr)

// Final Bands
var float upperBand = na
var float lowerBand = na

upperBand := na(upperBand[1]) ? upperBasic : (upperBasic < upperBand[1] or close[1] > upperBand[1]) ? upperBasic : upperBand[1]
lowerBand := na(lowerBand[1]) ? lowerBasic : (lowerBasic > lowerBand[1] or close[1] < lowerBand[1]) ? lowerBasic : lowerBand[1]

// Super Trend Direction
var int direction = 1
direction := na(direction[1]) ? 1 : 
             direction[1] == -1 and close > upperBand[1] ? 1 : 
             direction[1] == 1 and close < lowerBand[1] ? -1 : 
             direction[1]

// Super Trend Value
superTrend = direction == 1 ? lowerBand : upperBand

// Plotting
upTrend = direction == 1 ? superTrend : na
downTrend = direction == -1 ? superTrend : na

plot(upTrend, "Up Trend", color=color.green, linewidth=2, style=plot.style_linebr)
plot(downTrend, "Down Trend", color=color.red, linewidth=2, style=plot.style_linebr)

// Buy/Sell Signals
buySignal = direction == 1 and direction[1] == -1
sellSignal = direction == -1 and direction[1] == 1

// Buy/Sell Bubbles with Labels
plotshape(buySignal, "Buy", shape.circle, location.belowbar, color.green, size=size.normal, text="Buy", textcolor=color.white)
plotshape(sellSignal, "Sell", shape.circle, location.abovebar, color.red, size=size.normal, text="Sell", textcolor=color.white)

// Alerts
alertcondition(buySignal, "Super Trend Buy", "Super Trend flipped bullish")
alertcondition(sellSignal, "Super Trend Sell", "Super Trend flipped bearish")
```

# Trend Magic
A Pine Script implementation of the Trend Magic indicator with buy/sell signals.

## What is Trend Magic?

Trend Magic is a trend-following indicator that combines the Commodity Channel Index (CCI) for direction detection with Average True Range (ATR) for dynamic trailing levels. It plots a single line that trails price and changes color based on trend direction:

- **Green line below price** = Uptrend (bullish)
- **Red line above price** = Downtrend (bearish)

The indicator excels at identifying trend reversals earlier than purely price-based indicators by using momentum (CCI) as its directional trigger.

## How It Works

### Calculation

1. **CCI Calculation**: Measures price deviation from its statistical mean
2. **ATR Calculation**: Measures average price volatility over a specified period
3. **Trend Direction**:
   - CCI ≥ 0 = Bullish
   - CCI < 0 = Bearish
4. **Trend Magic Line**:
   - Uptrend: `Low - (ATR × Multiplier)` — ratchets upward only
   - Downtrend: `High + (ATR × Multiplier)` — ratchets downward only

### Signal Logic

| Signal | Condition |
|--------|-----------|
| **Buy** | CCI crosses above zero (trend flips bullish) |
| **Sell** | CCI crosses below zero (trend flips bearish) |

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| CCI Period | 20 | Lookback period for CCI calculation |
| ATR Period | 20 | Lookback period for ATR calculation |
| ATR Multiplier | 1.0 | Distance multiplier for the trailing line |

### Parameter Tuning

- **Higher CCI Period** = Smoother signals, fewer whipsaws, slower response
- **Lower CCI Period** = Faster signals, more noise, quicker response
- **Higher ATR Multiplier** = Line trails further from price, wider stops
- **Lower ATR Multiplier** = Line trails closer to price, tighter stops

## Features

- Overlay indicator (plots directly on price chart)
- Color-coded trend line (green = bullish, red = bearish)
- Visual buy/sell bubbles with labels at trend reversals
- Built-in alert conditions for automated notifications
- Ratcheting logic prevents premature signal flips

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

## Usage Notes

- Works on any timeframe and instrument
- CCI-based direction often catches reversals earlier than price-based methods
- Best used in trending markets; may generate false signals in extended ranges
- Consider combining with volume or other confirmation indicators
- Not financial advice—always backtest before live trading

```
//@version=5
indicator("Trend Magic", overlay=true)

// Inputs
cciPeriod = input.int(20, "CCI Period", minval=1)
atrPeriod = input.int(20, "ATR Period", minval=1)
atrMult = input.float(1.0, "ATR Multiplier", minval=0.1, step=0.1)

// CCI and ATR Calculation
cci = ta.cci(close, cciPeriod)
atr = ta.atr(atrPeriod)

// Trend Direction based on CCI
var int trend = 1
trend := cci >= 0 ? 1 : -1

// Trend Magic Line Calculation
upLine = low - (atr * atrMult)
downLine = high + (atr * atrMult)

var float trendMagic = na
trendMagic := trend == 1 ? math.max(nz(trendMagic[1]), upLine) : math.min(nz(trendMagic[1]), downLine)

// Reset on trend change
trendMagic := trend != trend[1] ? (trend == 1 ? upLine : downLine) : trendMagic

// Colors
lineColor = trend == 1 ? color.green : color.red

// Plot Trend Magic Line
plot(trendMagic, "Trend Magic", color=lineColor, linewidth=2, style=plot.style_line)

// Buy/Sell Signals
buySignal = trend == 1 and trend[1] == -1
sellSignal = trend == -1 and trend[1] == 1

// Buy/Sell Bubbles with Labels
plotshape(buySignal, "Buy", shape.circle, location.belowbar, color.green, size=size.normal, text="Buy", textcolor=color.white)
plotshape(sellSignal, "Sell", shape.circle, location.abovebar, color.red, size=size.normal, text="Sell", textcolor=color.white)

// Alerts
alertcondition(buySignal, "Trend Magic Buy", "Trend Magic flipped bullish")
alertcondition(sellSignal, "Trend Magic Sell", "Trend Magic flipped bearish")
```

