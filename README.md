# Project CWAP
This strategy is a multi-timeframe cross strategy on TradingView based on weighted average prices and developed by William Mutara. It is inspired by the already famous VWAP indiator.

# CWAP_LTF & CWAP_HTF Cross Strategy

## **Concept of the Strategy**

### **Weighted Average Prices:**
- **CWAP (Close-Weighted Average Price):** Measures the average price weighted by the closing price over a session.  
- **OWAP (Open-Weighted Average Price):** Similar to CWAP but weights the average based on the opening price.  
- **HWAP (High-Weighted Average Price):** Weights the average price based on the high price of the session.  
- **LWAP (Low-Weighted Average Price):** Weights the average price based on the low price of the session.  

### **Timeframes:**
- **LTF (Lower Timeframe):** Represents the immediate, short-term market sentiment.  
- **HTF (Higher Timeframe):** Provides a broader view of market momentum and trends.  

### **Signal Generation:**
- **Buy Signal:** When CWAP_HTF crosses above CWAP_LTF → indicates upward momentum.  
- **Sell Signal:** When CWAP_HTF crosses below CWAP_LTF → indicates downward momentum.  

---

## **How the Code Works**

### ✅ **1. New Day Check**
```pinescript
isNewDay = dayofmonth != dayofmonth[1]
```
- This checks if a new trading day has started.  
- If true, it resets the cumulative values for fresh calculations at the beginning of the day.  

---

### ✅ **2. CWAP (Close-Weighted Average Price) Calculation**
```pinescript
var float cumulativeWeightedClose = na
var float cumulativeClose = na

if isNewDay
    cumulativeWeightedClose := close * close
    cumulativeClose := close
else
    cumulativeWeightedClose := nz(cumulativeWeightedClose) + close * close
    cumulativeClose := nz(cumulativeClose) + close

cwap_ltf = cumulativeWeightedClose / cumulativeClose
```
**Logic:**  
- Multiplies the closing price by itself (squared) to give more weight to higher prices.  
- Cumulative values are summed up over the session.  
- The division gives a weighted average that reacts more to higher close prices.  

**Why Squaring?**  
- Squaring the price amplifies the effect of larger price moves, making the weighted average more sensitive to volatility.  
- This helps identify trend strength and momentum shifts more effectively than a simple average.  

---

### ✅ **3. OWAP, HWAP, and LWAP Calculation**
These calculations follow the same logic as CWAP but use the open, high, and low prices instead:

**OWAP (Open-Weighted Average Price):**
```pinescript
var float cumulativeWeightedOpen = na
var float cumulativeOpen = na

if isNewDay
    cumulativeWeightedOpen := open * open
    cumulativeOpen := open
else
    cumulativeWeightedOpen := nz(cumulativeWeightedOpen) + open * open
    cumulativeOpen := nz(cumulativeOpen) + open

owap_ltf = cumulativeWeightedOpen / cumulativeOpen
```

**HWAP (High-Weighted Average Price):**
```pinescript
var float cumulativeWeightedHigh = na
var float cumulativeHigh = na

if isNewDay
    cumulativeWeightedHigh := high * high
    cumulativeHigh := high
else
    cumulativeWeightedHigh := nz(cumulativeWeightedHigh) + high * high
    cumulativeHigh := nz(cumulativeHigh) + high

hwap_ltf = cumulativeWeightedHigh / cumulativeHigh
```

**LWAP (Low-Weighted Average Price):**
```pinescript
var float cumulativeWeightedLow = na
var float cumulativeLow = na

if isNewDay
    cumulativeWeightedLow := low * low
    cumulativeLow := low
else
    cumulativeWeightedLow := nz(cumulativeWeightedLow) + low * low
    cumulativeLow := nz(cumulativeLow) + low

lwap_ltf = cumulativeWeightedLow / cumulativeLow
```

---

### ✅ **4. Request Higher Timeframe (HTF) Values**
```pinescript
cwap_htf = request.security(syminfo.tickerid, "8", cwap_ltf, gaps=barmerge.gaps_on, lookahead=barmerge.lookahead_on)
```
- The `request.security` function pulls the CWAP, OWAP, HWAP, and LWAP values from a higher timeframe (`"8"` = 8-minute chart).  
- This allows the strategy to compare shorter-term (LTF) and longer-term (HTF) weighted averages.  

---

### ✅ **5. Plotting the Results**
```pinescript
plot(cwap_ltf, title="CWAP LTF", color=color.orange, linewidth=2)
plot(cwap_htf, title="CWAP HTF", color=color.red, linewidth=2)
```
- Plots the CWAP and OWAP values from both the lower and higher timeframes on the chart.  

```pinescript
candle_color = owap_htf < cwap_htf ? color.teal : color.red
plotcandle(owap_htf, hwap_htf, lwap_htf, cwap_htf, wickcolor = candle_color, bordercolor = candle_color, color = candle_color)
```
- Colors candles based on whether HTF OWAP is greater or lower than HTF CWAP — useful for identifying directional bias.  

---

### ✅ **6. Entry and Exit Conditions**
```pinescript
longCondition = ta.crossover(cwap_htf, cwap_ltf)
shortCondition = ta.crossunder(cwap_htf, cwap_ltf)
```
- **Long Condition:** When CWAP_HTF crosses above CWAP_LTF → bullish trend shift.  
- **Short Condition:** When CWAP_HTF crosses below CWAP_LTF → bearish trend shift.  

---

### ✅ **7. Trade Execution**
```pinescript
if longCondition
    strategy.entry("Long", strategy.long, qty=1)
    alert("Buy Alert!", alert.freq_once_per_bar_close)

if shortCondition
    strategy.entry("Short", strategy.short, qty=1)
    alert("Sell Alert!", alert.freq_once_per_bar_close)
```
- Executes a long or short trade when a crossover is detected.  
- Alerts are triggered to notify the user.  

---

## **Why This Strategy Works**
✅ **Multi-timeframe analysis:** Combining short-term and long-term signals helps filter out noise and improve signal accuracy.  
✅ **Weighted averaging by price strength:** Makes the signal more responsive to volatility and market momentum.  
✅ **Adaptive and modular:** The strategy is structured to be easy to tweak, optimize, and adapt to different market conditions.  

---

### 🔥 **Conclusion:**  
This strategy combines higher timeframe confirmation with lower timeframe execution, giving it the ability to react quickly to market changes while filtering out noise. By using weighted average price calculations, it captures momentum shifts and improves trade consistency.  

--- 
```
