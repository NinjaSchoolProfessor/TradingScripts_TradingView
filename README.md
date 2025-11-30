# TradingScripts_TradingView
Pine Scripts used with TradingView

# SuperTrend Indicator for TradingView

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

## Screenshots

*Add screenshots of the indicator on a chart here*

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
