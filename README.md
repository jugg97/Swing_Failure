# Swing Failure Pattern (SFP) Detector

**Point-in-Time Compliant Pattern Detection Module**

A research-grade Python module for detecting and visualizing Swing Failure Patterns in cryptocurrency and financial markets. Implements rigorous point-in-time compliance to ensure no look-ahead bias, making it suitable for both academic research and backtesting.

---

## 📋 Table of Contents

- [What are Swing Failure Patterns?](#what-are-swing-failure-patterns)
- [Pattern Definition](#pattern-definition)
- [How Detection Works](#how-detection-works)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Usage Examples](#usage-examples)
- [Visualization Guide](#visualization-guide)
- [Point-in-Time Compliance](#point-in-time-compliance)
- [API Reference](#api-reference)
- [Research Applications](#research-applications)
- [Contributing](#contributing)

---

## 🎯 What are Swing Failure Patterns?

**Swing Failure Patterns (SFPs)** are price action formations that occur when the market briefly breaks through a key swing level (high or low) but fails to sustain the move, quickly reversing direction. These patterns are significant because they often represent:

### Market Dynamics:
1. **Stop-Loss Hunting**: Large players trigger clustered stop-losses at obvious swing levels
2. **Liquidity Grabs**: Price moves to areas where liquidity is concentrated, then reverses
3. **Failed Breakouts**: Breakout attempts that trap retail traders before reversing
4. **Sentiment Shifts**: Rapid change in market participant behavior

### Real-World Significance:
- **For Traders**: High-probability reversal signals with clear risk/reward setups
- **For Researchers**: Evidence of order flow imbalances and microstructure dynamics
- **For Market Makers**: Insights into liquidity provision patterns

---

## 📐 Pattern Definition

### Bullish Swing Failure Pattern

A **Bullish SFP** occurs when:

```
1. A swing low (fractal low) is established
2. Price breaks BELOW that swing low (lower low made)
3. The SAME bar closes ABOVE the swing low (rejection)
4. This is the FIRST time this swing level has been breached
```

**Visual Example:**
```
                  ↓ Swing Low at $100
              |---●---|
              
Current bar:
    High:  $102
    Low:   $99    ← Breaks below $100 (stop hunt)
    Close: $101   ← Closes above $100 (reversal)
    
Result: ✅ Bullish SFP detected
```

**Interpretation**: 
- Sellers pushed price below support to trigger stops
- Buyers stepped in aggressively, closing above support
- Indicates potential upward reversal

---

### Bearish Swing Failure Pattern

A **Bearish SFP** occurs when:

```
1. A swing high (fractal high) is established
2. Price breaks ABOVE that swing high (higher high made)
3. The SAME bar closes BELOW the swing high (rejection)
4. This is the FIRST time this swing level has been breached
```

**Visual Example:**
```
  ↓ Swing High at $110
|---●---|
              
Current bar:
    High:  $111   ← Breaks above $110 (liquidity grab)
    Low:   $108
    Close: $109   ← Closes below $110 (reversal)
    
Result: ✅ Bearish SFP detected
```

**Interpretation**:
- Buyers pushed price above resistance to trigger stops
- Sellers stepped in aggressively, closing below resistance
- Indicates potential downward reversal

---

## 🔍 How Detection Works

### Three-Stage Process:

```
Stage 1: Fractal Detection (Swing Points)
    ↓
Stage 2: SFP Pattern Matching
    ↓
Stage 3: Validation & Recording
```

---

### Stage 1: Fractal Detection

The module uses **Bill Williams Fractals** to identify swing points:

#### Fractal Low (Swing Low):
```python
# A bar is a fractal low if its low is lower than:
- 2 bars before it
- 1 bar before it
- 1 bar after it
- 2 bars after it

Example (5-bar window):
Bar:   i-2   i-1    i    i+1   i+2
Low:   102   101   [98]   99    100

Result: Bar i is a fractal low ✓
```

#### Fractal High (Swing High):
```python
# A bar is a fractal high if its high is higher than:
- 2 bars before it
- 1 bar before it
- 1 bar after it  
- 2 bars after it

Example (5-bar window):
Bar:   i-2   i-1    i    i+1   i+2
High:  108   109   [112]  110   109

Result: Bar i is a fractal high ✓
```

**Key Point**: Fractals are only confirmed **2 bars after** they form, ensuring point-in-time compliance.

---

### Stage 2: SFP Pattern Matching

For each bar, the algorithm:

#### For Bullish SFP:
```python
1. Look back at last N fractal lows (configurable via fractal_lookback)
2. For each fractal low:
   a. Check if current bar's LOW breaks below fractal
   b. Check if current bar's CLOSE recovers above fractal
   c. Verify this is the FIRST time the fractal was breached
3. If all conditions met → Mark as Bullish SFP
```

#### For Bearish SFP:
```python
1. Look back at last N fractal highs
2. For each fractal high:
   a. Check if current bar's HIGH breaks above fractal
   b. Check if current bar's CLOSE recovers below fractal
   c. Verify this is the FIRST time the fractal was breached
3. If all conditions met → Mark as Bearish SFP
```

**Critical Validation**: The "first time breach" check ensures we detect genuine stop hunts, not re-tests of already broken levels.

---

### Stage 3: Validation & Recording

When a pattern is detected, the module records:

```python
{
    'sfp_bullish': True/False,           # Pattern type flag
    'sfp_bearish': True/False,           # Pattern type flag
    'sfp_fractal_price': 100.50,         # The swing level tested
    'sfp_fractal_bar': timestamp,        # When the swing formed
    'sfp_fractal_age': 15                # Bars ago the swing formed
}
```

This metadata enables:
- Pattern analysis (how old are swings when tested?)
- Performance studies (do fresh swings work better?)
- Microstructure research (add order flow metrics here)

---

## 💻 Installation

### Requirements

```bash
# Core dependencies
pip install pandas numpy plotly

# Optional (for data loading)
pip install ccxt  # For crypto exchange data
```

### Setup

```bash
# Clone repository
git clone https://github.com/yourusername/sfp-detector.git
cd sfp-detector

# Install dependencies
pip install -r requirements.txt

# Test installation
python sfp_detector.py
```

---

## 🚀 Quick Start

### Basic Usage (5 Lines)

```python
from sfp_detector import SFPDetector
import pandas as pd

# Load your OHLCV data
df = pd.read_csv('btc_5m.csv', index_col=0, parse_dates=True)

# Initialize detector
detector = SFPDetector(fractal_lookback=10)

# Run detection
detector.load_data(df)
results = detector.run_detection()

# Visualize
fig = detector.plot_interactive_chart()
fig.show()
```

**Expected Output:**
```
============================================================
SWING FAILURE PATTERN DETECTION
============================================================

Step 1: Detecting fractals...
✓ Fractals detected: 245 lows, 238 highs

Step 2: Detecting swing failure patterns...
✓ SFP patterns detected: 12 bullish, 8 bearish
  Bullish SFP avg fractal age: 23.4 bars
  Bearish SFP avg fractal age: 18.7 bars

============================================================
DETECTION COMPLETE
============================================================
```

---

## 📊 Usage Examples

### Example 1: Basic Detection

```python
from sfp_detector import SFPDetector
import pandas as pd

# Load data (must have DatetimeIndex and OHLC columns)
df = pd.read_csv('eth_15m.csv', index_col='timestamp', parse_dates=True)

# Initialize with 10-fractal lookback
detector = SFPDetector(fractal_lookback=10)

# Load and process
detector.load_data(df)
data_with_patterns = detector.run_detection()

# Access detected patterns
print(f"Total bars analyzed: {len(data_with_patterns)}")
print(f"Bullish SFPs found: {data_with_patterns['sfp_bullish'].sum()}")
print(f"Bearish SFPs found: {data_with_patterns['sfp_bearish'].sum()}")
```

---

### Example 2: Pattern Analysis

```python
# Get detailed pattern summary
summary = detector.get_pattern_summary()
print(summary)

# Output:
#                         Open    High     Low   Close  Fractal_Price  Fractal_Age_Bars    Type
# 2024-10-15 10:30:00  42150  42200  42050  42180         42075                15     Bullish
# 2024-10-15 14:45:00  42500  42650  42450  42480         42600                22     Bearish
# ...

# Analyze fractal age distribution
import matplotlib.pyplot as plt

bullish = summary[summary['Type'] == 'Bullish']
bearish = summary[summary['Type'] == 'Bearish']

plt.hist(bullish['Fractal_Age_Bars'], alpha=0.5, label='Bullish', bins=20)
plt.hist(bearish['Fractal_Age_Bars'], alpha=0.5, label='Bearish', bins=20)
plt.xlabel('Fractal Age (bars)')
plt.ylabel('Frequency')
plt.legend()
plt.title('SFP Fractal Age Distribution')
plt.show()
```

---

### Example 3: Multiple Timeframes

```python
# Analyze same asset on different timeframes
timeframes = ['5m', '15m', '1h']
results = {}

for tf in timeframes:
    df = load_data(f'btc_{tf}.csv')
    
    detector = SFPDetector(fractal_lookback=10)
    detector.load_data(df)
    data = detector.run_detection()
    
    results[tf] = {
        'bullish': data['sfp_bullish'].sum(),
        'bearish': data['sfp_bearish'].sum()
    }

print("Patterns by Timeframe:")
for tf, counts in results.items():
    print(f"{tf}: {counts['bullish']} bullish, {counts['bearish']} bearish")
```

---

### Example 4: Export for Further Analysis

```python
# Run detection
detector = SFPDetector(fractal_lookback=10)
detector.load_data(df)
data = detector.run_detection()

# Export patterns to CSV
detector.export_patterns_to_csv('sfp_patterns.csv')

# Or get full dataset with all flags
full_data = detector.data
full_data.to_csv('btc_with_sfp_flags.csv')

# Now you can add microstructure metrics:
# - VPIN at each SFP bar
# - Order book imbalance
# - Liquidation volume
# etc.
```

---

### Example 5: Parameter Sensitivity Analysis

```python
# Test different lookback windows
lookback_values = [3, 5, 10, 15, 20]
sensitivity_results = []

for lookback in lookback_values:
    detector = SFPDetector(fractal_lookback=lookback)
    detector.load_data(df)
    data = detector.run_detection()
    
    sensitivity_results.append({
        'lookback': lookback,
        'total_patterns': data['sfp_bullish'].sum() + data['sfp_bearish'].sum(),
        'avg_fractal_age': data[data['sfp_bullish'] | data['sfp_bearish']]['sfp_fractal_age'].mean()
    })

# Analyze: Does lookback affect pattern quality?
import pandas as pd
pd.DataFrame(sensitivity_results)
```

---

## 📈 Visualization Guide

### Interactive Chart Elements

When you call `detector.plot_interactive_chart()`, you get:

#### **Candlestick Chart**
- Green candles = price increase (close > open)
- Red candles = price decrease (close < open)
- Hover over any candle for OHLC details

#### **Fractal Markers**
- 🔵 **Blue triangles (pointing down)** = Fractal Lows (swing lows)
  - Position: Slightly below the bar's low
  - Hover to see: "Fractal Low - Price: XX.XX"
  
- 🟠 **Orange triangles (pointing up)** = Fractal Highs (swing highs)
  - Position: Slightly above the bar's high
  - Hover to see: "Fractal High - Price: XX.XX"

#### **SFP Markers**
- 🟢 **Green stars with ▲** = Bullish SFP
  - Position: Below the bar that formed the pattern
  - Hover to see: Low, Close, Fractal Price, Fractal Age
  
- 🔴 **Red stars with ▼** = Bearish SFP
  - Position: Above the bar that formed the pattern
  - Hover to see: High, Close, Fractal Price, Fractal Age

#### **Connection Lines**
- **Dashed lines** connect each SFP to its originating fractal
  - Green dashed = Bullish SFP to its fractal low
  - Red dashed = Bearish SFP to its fractal high
  - Shows which swing level was tested

---

### Chart Controls

The interactive Plotly chart includes:

```
🔍 Zoom: Click and drag on chart
📍 Pan: Hold Shift + drag
🏠 Reset: Double-click on chart
📷 Save: Camera icon (top-right) exports as PNG
🔽 Download: Download icon saves as HTML
```

---

### Customization Options

```python
# Custom chart dimensions
fig = detector.plot_interactive_chart(
    height=1200,        # Taller chart
    width=1600          # Wider chart
)

# Hide all fractals (show only SFPs)
fig = detector.plot_interactive_chart(
    show_all_fractals=False
)

# Custom title
fig = detector.plot_interactive_chart(
    title="BTC/USDT 5m - SFP Analysis Oct 2024"
)

# Combine options
fig = detector.plot_interactive_chart(
    height=1000,
    width=1800,
    show_all_fractals=True,
    title="ETH Swing Failure Patterns - Research Dataset"
)

fig.show()
```

---

## ⏱️ Point-in-Time Compliance

### Why It Matters

**Point-in-Time (PIT) compliance** ensures that at any given moment, you only use information that would have been available at that time. This is critical for:

1. **Academic Research**: Prevents inflated results from look-ahead bias
2. **Backtesting**: Ensures realistic performance estimates
3. **Live Trading**: Patterns detected in code match real-time reality
4. **Reproducibility**: Results can be independently verified

---

### How This Module Achieves PIT Compliance

#### **1. Fractal Confirmation Lag**

```python
# WRONG (look-ahead bias):
if bar[100].low < bar[101].low and bar[100].low < bar[102].low:
    mark_bar_100_as_fractal()  # ❌ Can't know this at bar 100!

# CORRECT (PIT compliant):
# At bar 102, we now can look back and confirm bar 100 was a fractal
if bar[100].low < bar[99].low and bar[100].low < bar[101].low and \
   bar[100].low < bar[101].low and bar[100].low < bar[102].low:
    mark_bar_100_as_fractal()  # ✅ Known at bar 102 (2 bars later)
```

**Implementation**:
```python
def calculate_fractals_pit(self, df):
    # Iterate through bars, but only confirm fractals using past data
    for i in range(2, len(df) - 2):
        # Check bar i using bars i-2, i-1, i+1, i+2
        # This simulates knowing at bar i+2 that bar i was a fractal
```

---

#### **2. Sequential SFP Detection**

```python
# For each current bar, only look BACKWARDS at fractals
for current_bar in all_bars:
    previous_fractals = get_fractals_before(current_bar)  # ✅ Historical only
    
    for fractal in previous_fractals:
        if is_sfp_pattern(current_bar, fractal):
            mark_sfp()
```

No future data is used in pattern detection.

---

#### **3. First-Time Breach Validation**

```python
# Check if fractal was previously breached
bars_between_fractal_and_current = get_bars_between(fractal_bar, current_bar)

# Only use PAST bars (those between fractal and current)
previously_breached = any(bar.close < fractal.price for bar in bars_between)

if previously_breached:
    return False  # Not a valid SFP (swing already broken)
```

This ensures we only detect patterns on the **first** time a swing is tested, using only historical data.

---

### Timing Diagram

```
Timeline of Events (PIT-Compliant):

Bar 100: Low = $98 (potential fractal)
Bar 101: Low = $99 (higher)
Bar 102: Low = $100 (higher) → NOW we know bar 100 was a fractal ✓

Bar 103: Can now check if this bar forms SFP against bar 100's fractal

Real-time trading timeline:
- At bar 100: Can't know it's a fractal yet
- At bar 101: Still can't confirm
- At bar 102: Fractal confirmed (2-bar lag)
- At bar 103+: Can detect SFPs against it
```

**Result**: No information from the future is used. All detection is realistic.

---

## 📚 API Reference

### `SFPDetector`

Main class for pattern detection.

#### Constructor

```python
SFPDetector(fractal_lookback: int = 10)
```

**Parameters:**
- `fractal_lookback` (int): Number of previous fractals to check when detecting SFPs
  - Lower values (3-5): Only recent swings, fewer but higher-quality patterns
  - Higher values (15-20): Older swings included, more patterns but potentially noisier

---

#### Methods

##### `load_data(df: pd.DataFrame) -> pd.DataFrame`

Load OHLCV data for analysis.

**Parameters:**
- `df`: DataFrame with DatetimeIndex and columns `['Open', 'High', 'Low', 'Close']`

**Returns:** Loaded and standardized DataFrame

**Example:**
```python
df = pd.read_csv('data.csv', index_col='timestamp', parse_dates=True)
detector.load_data(df)
```

---

##### `run_detection() -> pd.DataFrame`

Execute complete detection pipeline (fractals + SFPs).

**Returns:** DataFrame with all detection flags added

**Example:**
```python
results = detector.run_detection()
print(results.columns)
# ['open', 'high', 'low', 'close', 'fractal_low', 'fractal_high', 
#  'sfp_bullish', 'sfp_bearish', 'sfp_fractal_price', ...]
```

---

##### `get_pattern_summary() -> pd.DataFrame`

Get summary table of all detected patterns.

**Returns:** DataFrame with pattern details

**Example:**
```python
summary = detector.get_pattern_summary()
print(summary)
#                         Open    High     Low   Close  Fractal_Price  Fractal_Age_Bars    Type
# 2024-10-15 10:30:00  42150  42200  42050  42180         42075                15     Bullish
```

---

##### `plot_interactive_chart(...) -> go.Figure`

Create interactive visualization.

**Parameters:**
- `height` (int): Chart height in pixels (default: 900)
- `width` (int, optional): Chart width in pixels (default: auto)
- `show_all_fractals` (bool): Show all fractals or only SFP-related ones (default: True)
- `title` (str, optional): Custom chart title

**Returns:** Plotly Figure object

**Example:**
```python
fig = detector.plot_interactive_chart(height=1200, title="BTC Analysis")
fig.show()
fig.write_html('sfp_chart.html')  # Save as HTML
```

---

##### `export_patterns_to_csv(filename: str = 'sfp_patterns.csv') -> None`

Export detected patterns to CSV file.

**Parameters:**
- `filename` (str): Output CSV filename

**Example:**
```python
detector.export_patterns_to_csv('btc_patterns.csv')
```

---

### Data Schema

#### Input Data Format

```python
# Required columns (case-insensitive):
df = pd.DataFrame({
    'timestamp': [...],      # DatetimeIndex
    'open': [...],          # Opening price
    'high': [...],          # High price
    'low': [...],           # Low price
    'close': [...]          # Closing price
})
df.set_index('timestamp', inplace=True)
```

#### Output Data Columns

After `run_detection()`, DataFrame includes:

```python
# Original columns
'open', 'high', 'low', 'close'

# Fractal detection
'fractal_low'          # Boolean: Is this bar a fractal low?
'fractal_high'         # Boolean: Is this bar a fractal high?
'fractal_low_price'    # Float: Price of fractal low (if applicable)
'fractal_high_price'   # Float: Price of fractal high (if applicable)

# SFP detection
'sfp_bullish'          # Boolean: Is this a bullish SFP?
'sfp_bearish'          # Boolean: Is this a bearish SFP?
'sfp_fractal_price'    # Float: The swing level that was tested
'sfp_fractal_bar'      # Timestamp: When the fractal formed
'sfp_fractal_age'      # Int: How many bars ago the fractal formed
```

---

## 🔬 Research Applications

### Adding Microstructure Metrics

This module is designed as the **first step** in microstructure research. After detecting patterns, add your metrics:

```python
# Detect patterns
detector = SFPDetector(fractal_lookback=10)
detector.load_data(ohlcv_df)
data = detector.run_detection()

# Load your microstructure data (order book, trades, liquidations)
order_book_df = pd.read_parquet('orderbook_snapshots.parquet')
trades_df = pd.read_parquet('tick_trades.parquet')
liquidations_df = pd.read_csv('liquidations.csv')

# At each SFP bar, compute metrics
sfp_bars = data[data['sfp_bullish'] | data['sfp_bearish']].index

for sfp_time in sfp_bars:
    # Get order book state at SFP time
    book_at_sfp = order_book_df[order_book_df.index == sfp_time].iloc[0]
    
    # Compute order book imbalance
    data.loc[sfp_time, 'obi'] = compute_obi(book_at_sfp)
    
    # Get trades in 5-min window before SFP
    recent_trades = trades_df[
        (trades_df.index < sfp_time) &
        (trades_df.index >= sfp_time - pd.Timedelta(minutes=5))
    ]
    
    # Compute VPIN
    data.loc[sfp_time, 'vpin'] = compute_vpin(recent_trades)
    
    # Check for liquidations at swing level
    liq_at_level = liquidations_df[
        (liquidations_df.timestamp == sfp_time) &
        (abs(liquidations_df.price - data.loc[sfp_time, 'sfp_fractal_price']) < 10)
    ]
    data.loc[sfp_time, 'liquidation_volume'] = liq_at_level['quantity'].sum()

# Now run your analysis
sfp_data = data[data['sfp_bullish'] | data['sfp_bearish']].copy()
print(f"Average VPIN at SFP: {sfp_data['vpin'].mean()}")
print(f"Average OBI at SFP: {sfp_data['obi'].mean()}")
```

---

### Hypothesis Testing Examples

```python
# H1: Liquidity withdrawal before SFPs
# Compare order book depth before SFP vs baseline
baseline_depth = data['depth'].mean()
sfp_depth = data[data['sfp_bullish']]['depth'].mean()

from scipy.stats import ttest_1samp
t_stat, p_value = ttest_1samp(
    data[data['sfp_bullish']]['depth'], 
    baseline_depth
)
print(f"Depth at bullish SFP vs baseline: t={t_stat:.2f}, p={p_value:.4f}")

# H2: Order flow imbalance predicts SFP success
# Define "successful" SFP as price moving expected direction within 20 bars
def sfp_success(row, data, bars_forward=20):
    idx = data.index.get_loc(row.name)
    future_bars = data.iloc[idx:idx+bars_forward]
    
    if row['sfp_bullish']:
        return (future_bars['close'].max() - row['close']) > 0
    elif row['sfp_bearish']:
        return (row['close'] - future_bars['close'].min()) > 0
    return False

sfp_data['success'] = sfp_data.apply(lambda row: sfp_success(row, data), axis=1)

# Compare VPIN for successful vs failed SFPs
successful = sfp_data[sfp_data['success']]
failed = sfp_data[~sfp_data['success']]

print(f"VPIN - Successful SFPs: {successful['vpin'].mean():.4f}")
print(f"VPIN - Failed SFPs: {failed['vpin'].mean():.4f}")
```

---

## 🤝 Contributing

We welcome contributions! Areas for improvement:

### Enhancements Needed:
- [ ] Add more fractal types (Donchian channels, pivot points)
- [ ] Implement volume-weighted fractals
- [ ] Add risk/reward calculation for each pattern
- [ ] Multi-timeframe confirmation
- [ ] Pattern success rate tracking
- [ ] Integration with popular data providers (ccxt, yfinance)

### How to Contribute:

```bash
# 1. Fork the repository
# 2. Create feature branch
git checkout -b feature/your-feature-name

# 3. Make changes
# 4. Test thoroughly
python -m pytest tests/

# 5. Commit with clear messages
git commit -m "Add: Description of your feature"

# 6. Push and create Pull Request
git push origin feature/your-feature-name
```

---

## 📖 References

### Academic Papers on Market Microstructure:
- Easley, D., López de Prado, M., & O'Hara, M. (2012). "Flow Toxicity and Liquidity in a High Frequency World"
- Heston, S., Korajczyk, R., & Sadka, R. (2010). "Intraday Patterns in the Cross-Section of Stock Returns"
- Kyle, A. S. (1985). "Continuous Auctions and Insider Trading"

### Trading Literature:
- Williams, B. (1995). "Trading Chaos: Maximize Profits with Proven Technical Techniques"
- Concepts related to stop-hunting and liquidity provision in practitioner literature

---

## 📄 License

MIT License - see LICENSE file for details

---

## 💬 Support

- **Issues**: Report bugs at [GitHub Issues](https://github.com/yourusername/sfp-detector/issues)
- **Discussions**: Ask questions in [GitHub Discussions](https://github.com/yourusername/sfp-detector/discussions)
- **Email**: research@yourproject.com

---

## 🎓 Citation

If you use this module in your research, please cite:

```bibtex
@software{sfp_detector_2025,
  title = {Swing Failure Pattern Detector: Point-in-Time Compliant Pattern Detection},
  author = {Your Name},
  year = {2025},
  url = {https://github.com/yourusername/sfp-detector}
}
```

---

## ⚠️ Disclaimer

This software is for research and educational purposes only. Not financial advice. Trading cryptocurrencies and financial instruments involves substantial risk of loss. Past pattern performance does not guarantee future results.

---

**Last Updated**: January 2025  
**Version**: 1.0.0  
**Status**: Active Development

---

*Built with ❤️ for the quantitative trading and market microstructure research community*