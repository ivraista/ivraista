//@version=4
strategy("BRAHIM KHATTARA", overlay=true)

rsiOS = 30
rsiOB = 70

// Custom Strategy Conditions
rsi = rsi(close, 14)

upTrend = change(rsi) >= 0
downTrend = change(rsi) <= 0

oversold = false
overbought = false

if rsi >= rsiOS and rsi[1] < rsiOS 
    oversold := true
else if rsi <= rsiOB and rsi[1] > rsiOB 
    overbought := true

// Define the showAllExtremes variable
showAllExtremes = true // or false, depending on your requirements

// Plot Buy and Sell Shapes
plotshape(upTrend and not showAllExtremes and oversold ? close : na, style=shape.triangleup, location=location.belowbar, size=size.small, title="Buy the dip", color=#87cefa)
plotshape(downTrend and not showAllExtremes and overbought ? close : na, style=shape.triangledown, location=location.abovebar, size=size.small, title="Sell the top", color=#87cefa)
plotshape(showAllExtremes and overbought ? close : na, style=shape.triangledown, location=location.abovebar, size=size.small, title="Sell the top", color=#87cefa)
plotshape(showAllExtremes and oversold ? close : na, style=shape.triangleup, location=location.belowbar, size=size.small, title="Buy the dip", color=#87cefa)

// Plot Buy and Sell Labels
var lastBuyLabelBar = 0
var lastSellLabelBar = 0

if oversold
    lastBuyLabelBar := bar_index
if overbought
    lastSellLabelBar := bar_index

plotshape(oversold, style=shape.labelup, location=location.belowbar, color=color.green, text="buy dip",textcolor=color.white, size=size.tiny, title="Sell label")
plotshape(overbought, style=shape.labeldown, location=location.abovebar, color=color.red, text="sell top",textcolor=color.white, size=size.tiny, title="Buy label")

// Calculate Label Location
label_X_Loc = time_close + (( time_close - time_close[1] ) * 2 ) // Set Label offset

// Display RSI and %R Values
col = color.white
txt_col = color.black

// Show RSI Labels conditionally
if showAllExtremes
    label.new(label_X_Loc, close, "RSI:" + tostring(round(rsi,2)) + "  (O/B>70, O/S<30)\n%R: " + tostring(round(rsi,2)) + "  (O/B>-20, O/S<-80)", xloc.bar_time, yloc.price, color=col, textcolor=txt_col, size=size.normal, textalign=text.align_left)

// Fibonacci Bollinger Bands Indicator
length = input(200, title="Length", minval=1)
src = input(hlc3, title="Source")
mult = input(3.0, title="Multiplier", minval=0.001, maxval=50.0)
basis = vwma(src, length)
dev = mult * stdev(src, length)

upper_1 = basis + (0.236 * dev)
upper_2 = basis + (0.382 * dev)
upper_3 = basis + (0.5 * dev)
upper_4 = basis + (0.618 * dev)
upper_5 = basis + (0.764 * dev)
upper_6 = basis + dev // Define upper_6 correctly

lower_1 = basis - (0.236 * dev)
lower_2 = basis - (0.382 * dev)
lower_3 = basis - (0.5 * dev)
lower_4 = basis - (0.618 * dev)
lower_5 = basis - (0.764 * dev)
lower_6 = basis - dev // Define lower_6 correctly

plot(basis, color=color.fuchsia, linewidth=2)
p1 = plot(upper_1, color=color.white, linewidth=1, title="0.236")
p2 = plot(upper_2, color=color.white, linewidth=1, title="0.382")
p3 = plot(upper_3, color=color.white, linewidth=1, title="0.5")
p4 = plot(upper_4, color=color.white, linewidth=1, title="0.618")
p5 = plot(upper_5, color=color.white, linewidth=1, title="0.764")
p6 = plot(upper_6, color=color.red, linewidth=2, title="1")
p13 = plot(lower_1, color=color.white, linewidth=1, title="0.236")
p14 = plot(lower_2, color=color.white, linewidth=1, title="0.382")
p15 = plot(lower_3, color=color.white, linewidth=1, title="0.5")
p16 = plot(lower_4, color=color.white, linewidth=1, title="0.618")
p17 = plot(lower_5, color=color.white, linewidth=1, title="0.764")
p18 = plot(lower_6, color=color.green, linewidth=2, title="1")

// Define Stop Loss Distance (in ticks)
stopLossDistance = input(100, title="Stop Loss Distance (in ticks)")

// Strategy Entry Conditions
longCondition = not na(lastBuyLabelBar) and (na(lastSellLabelBar) or lastBuyLabelBar > lastSellLabelBar)
shortCondition = not na(lastSellLabelBar) and (na(lastBuyLabelBar) or lastSellLabelBar > lastBuyLabelBar)

// Place Buy and Sell Orders with Stop Loss
strategy.entry("Long", strategy.long, when=longCondition)
strategy.entry("Short", strategy.short, when=shortCondition)

// Set Stop Loss for Long and Short Trades
strategy.exit("Long Stop Loss", from_entry="Long", when=longCondition, trail_offset=stopLossDistance)
strategy.exit("Short Stop Loss", from_entry="Short", when=shortCondition, trail_offset=stopLossDistance)

// PLOT Selling Signals on Chart
plotshape(shortCondition, title="Short Sell Signal", color=color.red, style=shape.triangledown, location=location.abovebar, size=size.small, title="Short Sell")
plotshape(longCondition, title="Long Buy Signal", color=color.green, style=shape.triangleup, location=location.belowbar, size=size.small, title="Long Buy")
