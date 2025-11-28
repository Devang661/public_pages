
K线是最不滞后的，其次是RSI-14的斜率，NSM就是相对滞后的，均线也是
一.【做多】

1，移动止损指标发出买入信号；

2，关键K线：收盘价位于均线之上的阳线；

3，止损：关键K线低点下方，或者阶段性低点下方；

4，止盈：收盘价走出下穿均线的K线。
二.【做空】
1，移动止损指标发出卖出信号；

2，关键K线：收盘价位于均线之下的阴线；

3，止损：关键K线高点上方，或者阶段性高点上方；

4，止盈：收盘价走出上穿均线的K线。


## 初级 ZLSMA + Chandelier Exit 
<img width="1205" height="591" alt="image" src="https://github.com/user-attachments/assets/b28597ac-ec4e-49d2-8349-41f76c17ccc2" />


<img width="825" height="820" alt="image" src="https://github.com/user-attachments/assets/b706b03d-ca2b-499a-a93e-c99f95674981" />

```m
//@version=6
indicator("ZLSMA + Chandelier Exit (Merged, Fixed)", shorttitle="ZLSMA+CE", overlay=true)


groupZ = "ZLSMA Settings"

lengthZ = input.int(32, title="ZLSMA Length", group=groupZ)
offsetZ = input.int(0, title="ZLSMA Offset", group=groupZ)
srcZ = input(close, title="ZLSMA Source", group=groupZ)

lsma = ta.linreg(srcZ, lengthZ, offsetZ)
lsma2 = ta.linreg(lsma, lengthZ, offsetZ)
eq = lsma - lsma2
zlsma = lsma + eq

plot(zlsma, title="ZLSMA", color=color.yellow, linewidth=3)

//─────────────────────────
// Chandelier Exit (修复变量声明部分)
//─────────────────────────
groupCEcalc = "CE Calculation"
groupCEvis = "CE Visual"
groupCEalert = "CE Alerts"

length = input.int(22, title="ATR Period", group=groupCEcalc)
mult = input.float(3.0, step=0.1, title="ATR Multiplier", group=groupCEcalc)
useClose = input.bool(true, title="Use Close Price for Extremums", group=groupCEcalc)

showLabels = input.bool(true, title="Show Buy/Sell Labels", group=groupCEvis)
highlightState = input.bool(true, title="Highlight State", group=groupCEvis)

awaitBarConfirmation = input.bool(true, title="Await Bar Confirmation", group=groupCEalert)

atr = mult * ta.atr(length)

//─────────────────────────
// ★ 必须先声明变量 ★
//─────────────────────────
var float longStop = na
var float shortStop = na

//─────────────────────────
// Long Stop Calculation
//─────────────────────────
longStopRaw = (useClose ? ta.highest(close, length) : ta.highest(length)) - atr
longStopPrev = nz(longStop[1], longStopRaw)
longStop := close[1] > longStopPrev ? math.max(longStopRaw, longStopPrev) : longStopRaw

//─────────────────────────
// Short Stop Calculation
//─────────────────────────
shortStopRaw = (useClose ? ta.lowest(close, length) : ta.lowest(length)) + atr
shortStopPrev = nz(shortStop[1], shortStopRaw)
shortStop := close[1] < shortStopPrev ? math.min(shortStopRaw, shortStopPrev) : shortStopRaw

//─────────────────────────
// Direction
//─────────────────────────
var int dir = 1
dir := close > shortStopPrev ? 1 : close < longStopPrev ? -1 : dir

//─────────────────────────
// Colors
//─────────────────────────
textColor = color.white
longColor = color.green
shortColor = color.red
longFillColor = color.new(color.green, 85)
shortFillColor = color.new(color.red, 85)

//─────────────────────────
// Signals
//─────────────────────────
buySignal = dir == 1 and dir[1] == -1
sellSignal = dir == -1 and dir[1] == 1

//─────────────────────────
// Plot CE
//─────────────────────────
longStopPlot = plot(dir == 1 ? longStop : na, title="Long Stop", style=plot.style_linebr, linewidth=2, color=longColor)
plotshape(buySignal ? longStop : na, title="Long Stop Start", location=location.absolute, style=shape.circle, size=size.tiny, color=longColor)
plotshape(buySignal and showLabels ? longStop : na, title="Buy Label", text="Buy", location=location.absolute, style=shape.labelup, size=size.tiny, color=longColor, textcolor=textColor)

shortStopPlot = plot(dir == 1 ? na : shortStop, title="Short Stop", style=plot.style_linebr, linewidth=2, color=shortColor)
plotshape(sellSignal ? shortStop : na, title="Short Stop Start", location=location.absolute, style=shape.circle, size=size.tiny, color=shortColor)
plotshape(sellSignal and showLabels ? shortStop : na, title="Sell Label", text="Sell", location=location.absolute, style=shape.labeldown, size=size.tiny, color=shortColor, textcolor=textColor)

midPricePlot = plot(ohlc4, title="", display=display.none, editable=false)

// Fill
fill(midPricePlot, longStopPlot, title="Long State Filling", color=(highlightState and dir == 1 ? longFillColor : na))
fill(midPricePlot, shortStopPlot, title="Short State Filling", color=(highlightState and dir == -1 ? shortFillColor : na))

// Alerts
await = awaitBarConfirmation ? barstate.isconfirmed : true
alertcondition(dir != dir[1] and await, title="CE Direction Change", message="Chandelier Exit Direction Change, {{exchange}}:{{ticker}}")
alertcondition(buySignal and await, title="CE Buy", message="Chandelier Exit Buy, {{exchange}}:{{ticker}}")
alertcondition(sellSignal and await, title="CE Sell", message="Chandelier Exit Sell, {{exchange}}:{{ticker}}")


```


## 升级 ZLSMA + Chandelier Exit  +ma


```
//@version=6
indicator("ZLSMA + Chandelier Exit + Double Moving Averages", shorttitle="ZLSMA+CE+MA", overlay=true)

//---------------------------------
// ZLSMA 部分
//---------------------------------
groupZ = "ZLSMA Settings"

lengthZ = input.int(32, title="ZLSMA Length", group=groupZ)
offsetZ = input.int(0, title="ZLSMA Offset", group=groupZ)
srcZ = input(close, title="ZLSMA Source", group=groupZ)

lsma = ta.linreg(srcZ, lengthZ, offsetZ)
lsma2 = ta.linreg(lsma, lengthZ, offsetZ)
eq = lsma - lsma2
zlsma = lsma + eq

plot(zlsma, title="ZLSMA", color=color.yellow, linewidth=3)

//---------------------------------
// 双均线策略
//---------------------------------
groupMA = "Moving Averages Settings"

smaLength20 = input.int(20, title="SMA 20 Length", group=groupMA)
smaLength60 = input.int(60, title="SMA 60 Length", group=groupMA)
smaLength120 = input.int(120, title="SMA 120 Length", group=groupMA)

emaLength20 = input.int(20, title="EMA 20 Length", group=groupMA)
emaLength60 = input.int(60, title="EMA 60 Length", group=groupMA)
emaLength120 = input.int(120, title="EMA 120 Length", group=groupMA)

// 计算 SMA 和 EMA
sma20 = ta.sma(close, smaLength20)
sma60 = ta.sma(close, smaLength60)
sma120 = ta.sma(close, smaLength120)

ema20 = ta.ema(close, emaLength20)
ema60 = ta.ema(close, emaLength60)
ema120 = ta.ema(close, emaLength120)

// 绘制 SMA 和 EMA
plot(sma20, title="SMA 20", color=color.black, linewidth=2)
plot(sma60, title="SMA 60", color=color.blue, linewidth=2)
plot(sma120, title="SMA 120", color=color.purple, linewidth=2)

plot(ema20, title="EMA 20", color=color.new(color.black, 0), linewidth=2)
plot(ema60, title="EMA 60", color=color.new(color.blue, 0), linewidth=2)
plot(ema120, title="EMA 120", color=color.new(color.purple, 0), linewidth=2)

//---------------------------------
// Chandelier Exit 部分
//---------------------------------
groupCEcalc = "CE Calculation"
groupCEvis = "CE Visual"
groupCEalert = "CE Alerts"

length = input.int(22, title="ATR Period", group=groupCEcalc)
mult = input.float(3.0, step=0.1, title="ATR Multiplier", group=groupCEcalc)
useClose = input.bool(true, title="Use Close Price for Extremums", group=groupCEcalc)

showLabels = input.bool(true, title="Show Buy/Sell Labels", group=groupCEvis)
highlightState = input.bool(true, title="Highlight State", group=groupCEvis)

awaitBarConfirmation = input.bool(true, title="Await Bar Confirmation", group=groupCEalert)

atr = mult * ta.atr(length)

//─────────────────────────
// 必须先声明变量
//─────────────────────────
var float longStop = na
var float shortStop = na

//─────────────────────────
// Long Stop Calculation
//─────────────────────────
longStopRaw = (useClose ? ta.highest(close, length) : ta.highest(length)) - atr
longStopPrev = nz(longStop[1], longStopRaw)
longStop := close[1] > longStopPrev ? math.max(longStopRaw, longStopPrev) : longStopRaw

//─────────────────────────
// Short Stop Calculation
//─────────────────────────
shortStopRaw = (useClose ? ta.lowest(close, length) : ta.lowest(length)) + atr
shortStopPrev = nz(shortStop[1], shortStopRaw)
shortStop := close[1] < shortStopPrev ? math.min(shortStopRaw, shortStopPrev) : shortStopRaw

//─────────────────────────
// Direction
//─────────────────────────
var int dir = 1
dir := close > shortStopPrev ? 1 : close < longStopPrev ? -1 : dir

//─────────────────────────
// Colors
//─────────────────────────
textColor = color.white
longColor = color.green
shortColor = color.red
longFillColor = color.new(color.green, 85)
shortFillColor = color.new(color.red, 85)

//─────────────────────────
// Signals
//─────────────────────────
buySignal = dir == 1 and dir[1] == -1
sellSignal = dir == -1 and dir[1] == 1

//─────────────────────────
// Plot CE
//─────────────────────────
longStopPlot = plot(dir == 1 ? longStop : na, title="Long Stop", style=plot.style_linebr, linewidth=2, color=longColor)
plotshape(buySignal ? longStop : na, title="Long Stop Start", location=location.absolute, style=shape.circle, size=size.tiny, color=longColor)
plotshape(buySignal and showLabels ? longStop : na, title="Buy Label", text="Buy", location=location.absolute, style=shape.labelup, size=size.tiny, color=longColor, textcolor=textColor)

shortStopPlot = plot(dir == 1 ? na : shortStop, title="Short Stop", style=plot.style_linebr, linewidth=2, color=shortColor)
plotshape(sellSignal ? shortStop : na, title="Short Stop Start", location=location.absolute, style=shape.circle, size=size.tiny, color=shortColor)
plotshape(sellSignal and showLabels ? shortStop : na, title="Sell Label", text="Sell", location=location.absolute, style=shape.labeldown, size=size.tiny, color=shortColor, textcolor=textColor)

midPricePlot = plot(ohlc4, title="", display=display.none, editable=false)

// Fill
fill(midPricePlot, longStopPlot, title="Long State Filling", color=(highlightState and dir == 1 ? longFillColor : na))
fill(midPricePlot, shortStopPlot, title="Short State Filling", color=(highlightState and dir == -1 ? shortFillColor : na))

// Alerts
await = awaitBarConfirmation ? barstate.isconfirmed : true
alertcondition(dir != dir[1] and await, title="CE Direction Change", message="Chandelier Exit Direction Change, {{exchange}}:{{ticker}}")
alertcondition(buySignal and await, title="CE Buy", message="Chandelier Exit Buy, {{exchange}}:{{ticker}}")
alertcondition(sellSignal and await, title="CE Sell", message="Chandelier Exit Sell, {{exchange}}:{{ticker}}")

```



