# Rules for Adding New Trading Strategies

## 1. Strategy File Structure

### Required Functions
Every strategy file must implement these functions:

```python
def detect_signals(df, **params):
    """
    Detect trading signals from price data
    Returns: DataFrame with columns ['type', 'signal_date', 'entry_price', ...]
    """
    pass

def execute_backtest(df, signals, **params):
    """
    Execute backtest with risk management
    Returns: DataFrame with columns ['type', 'entry_date', 'entry', 'stop', 'R', 'outcome_R']
    """
    pass

def summarize_results(trades):
    """
    Calculate summary metrics
    Returns: Dict with keys ['num_trades', 'avg_outcome_R', 'win_rate_pos_R', 'bullish_trades', 'bearish_trades']
    """
    pass
```

### Required Parameters
- `df`: DataFrame with columns ['open', 'high', 'low', 'close', 'volume']
- `**params`: Strategy-specific parameters (e.g., lookback periods, thresholds)

## 2. Integration Steps

### Step 1: Create Strategy Runner
Add to `run_backtests.py`:

```python
def run_strategy_backtest(data, symbol_name, strategy_name="new_strategy"):
    """Run any strategy on symbol data"""
    try:
        # Import strategy module
        strategy_module = __import__(f"{strategy_name}")
        
        # Convert data and run strategy
        df = convert_json_to_dataframe(data)
        signals = strategy_module.detect_signals(df)
        trades = strategy_module.execute_backtest(df, signals)
        summary = strategy_module.summarize_results(trades)
        
        return format_strategy_results(symbol_name, summary, trades, strategy_name)
    except Exception as e:
        return handle_strategy_error(symbol_name, strategy_name, e)
```

### Step 2: Update Web API
Add to `web_app.py`:

```python
@app.route('/api/strategy-results/<strategy_name>')
def api_strategy_results(strategy_name):
    try:
        with open(f'cache/{strategy_name}_results.json', 'r') as f:
            results = json.load(f)
        return jsonify(results)
    except Exception as e:
        return jsonify({'error': str(e)}), 500
```

### Step 3: Add UI Components
Update `backtest_results.html`:

```html
<!-- Add strategy selector -->
<div class="strategy-selector">
    <label>Strategy:</label>
    <select id="strategySelect" onchange="switchStrategy()">
        <option value="ob_refined">Order Block Strategy</option>
        <option value="new_strategy">New Strategy</option>
    </select>
</div>
```

## 3. Data Format Standards

### Input Data Format
```python
{
    'dates': ['2023-01-01', '2023-01-02', ...],
    'prices': {
        'open': [100.0, 101.0, ...],
        'high': [102.0, 103.0, ...],
        'low': [99.0, 100.0, ...],
        'close': [101.0, 102.0, ...],
        'volume': [1000000, 1100000, ...]
    }
}
```

### Output Trade Format
```python
{
    'type': 'Bullish' | 'Bearish',
    'entry_date': '2023-01-01',
    'entry': 100.50,
    'stop': 99.00,
    'R': 1.50,
    'outcome_R': 2.0
}
```

### Summary Format
```python
{
    'num_trades': 25,
    'avg_outcome_R': 0.75,
    'win_rate_pos_R': 0.68,
    'bullish_trades': 15,
    'bearish_trades': 10
}
```

## 4. File Naming Conventions

### Strategy Files
- `{strategy_name}_strategy.py` - Main strategy implementation
- `{strategy_name}_config.json` - Strategy parameters
- `{strategy_name}_results.json` - Cached backtest results

### Cache Files
- `cache/{strategy_name}_results.json` - Backtest results
- `src/{strategy_name}_results.json` - Web app copy

## 5. UI Integration Rules

### Strategy Selector
Add strategy to dropdown in all relevant pages:
- `backtest_results.html`
- `backtest_detail.html`
- `interactive_dashboard.html` (if applicable)

### URL Structure
- `/backtests/{strategy_name}` - Strategy overview
- `/backtest/{strategy_name}/{data_type}/{symbol}` - Detailed analysis

### JavaScript Functions
Required functions for each strategy:
```javascript
function loadStrategyResults(strategyName) { ... }
function renderStrategyCharts(strategyName, data) { ... }
function switchStrategy() { ... }
```

## 6. Error Handling Standards

### Strategy Errors
```python
def handle_strategy_error(symbol, strategy, error):
    return {
        'symbol': symbol,
        'strategy': strategy,
        'error': str(error),
        'summary': {
            'num_trades': 0,
            'avg_outcome_R': 0.0,
            'win_rate_pos_R': 0.0,
            'bullish_trades': 0,
            'bearish_trades': 0
        }
    }
```

### UI Error Display
```javascript
function displayStrategyError(error) {
    document.getElementById('resultsGrid').innerHTML = 
        `<div class="loading">Strategy Error: ${error}</div>`;
}
```

## 7. Performance Requirements

### Execution Time
- Strategy execution: < 5 seconds per symbol
- UI updates: < 1 second after strategy switch
- Cache loading: < 500ms

### Memory Usage
- Keep DataFrames under 100MB per symbol
- Clear unused strategy data after switching

## 8. Testing Checklist

### Before Integration
- [ ] Strategy runs on all asset types (stocks, forex, commodities)
- [ ] Returns valid trade format
- [ ] Handles edge cases (no signals, all losses, etc.)
- [ ] Performance within limits

### After Integration
- [ ] UI loads strategy results
- [ ] Charts render correctly
- [ ] Admin settings work (capital/risk adjustment)
- [ ] Detailed pages accessible
- [ ] No JavaScript errors in console

## 9. Configuration Management

### Strategy Parameters
Store in `{strategy_name}_config.json`:
```json
{
    "name": "New Strategy",
    "description": "Strategy description",
    "parameters": {
        "lookback": 20,
        "threshold": 0.02,
        "risk_reward": 2.0
    },
    "ui_settings": {
        "color": "#ff6b6b",
        "icon": "📊"
    }
}
```

## 10. Deployment Steps

### Step-by-Step Integration
1. Create strategy file following format
2. Test strategy on sample data
3. Add to `run_backtests.py`
4. Generate results cache
5. Update web API routes
6. Add UI components
7. Test full integration
8. Deploy and verify

### Rollback Plan
- Keep previous strategy files
- Maintain separate cache files
- Use feature flags for UI components
- Test rollback procedure

## 11. Documentation Requirements

### Strategy Documentation
Each strategy must include:
- Algorithm description
- Parameter explanations
- Expected performance characteristics
- Risk considerations
- Usage examples

### Code Comments
```python
def detect_signals(df, lookback=20, threshold=0.02):
    """
    Detect trading signals using [algorithm name]
    
    Args:
        df: OHLCV DataFrame
        lookback: Period for calculation (default: 20)
        threshold: Signal threshold (default: 0.02)
    
    Returns:
        DataFrame with signal information
    """
```

## 12. Maintenance Guidelines

### Regular Updates
- Review strategy performance monthly
- Update parameters based on market conditions
- Archive outdated strategies
- Monitor execution times

### Version Control
- Tag strategy versions
- Maintain changelog
- Keep configuration history
- Document parameter changes

---

**Follow these rules to ensure seamless strategy integration without breaking existing functionality.**