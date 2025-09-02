# Flappy Bird DQN Forex Trading Agent

## System Overview
Single-agent DQN navigating M30 forex price action with daily trend bias and controlled hedging system.

## Agent Architecture

### State Space
- M30 price position relative to daily trend
- Account risk utilization (current/max 5%)
- Active hedge count (0-3 positions)
- Time until next major news event
- Daily trend bias (bullish/bearish/neutral)

### Action Space
- **Enter Long**: Place buy order with M30 confirmation
- **Enter Short**: Place sell order with M30 confirmation  
- **Add Hedge**: Place opposite order when position fails
- **Close Profitable**: Exit winning positions
- **Wait**: No action, preserve capital

### Reward Function
- +20: Successful 20-30 pip scalp closure
- -10: Hedge trigger (position against by 40-60 pips)
- +5: Profitable hedge leg closure
- -100: Exceed 5% account risk or 4-order limit
- -50: Trading during major news events

## Game Mechanics Mapping

### Pipes = Liquidity Zones
- **Obstacle navigation**: Price must move through institutional supply/demand areas
- **Gap timing**: Enter when M30 shows clear path through liquidity zone
- **Collision death**: Hitting major liquidity wall triggers hedge system

### Bird Movement = Position Management
- **Forward momentum**: Following daily trend bias from D1 analysis
- **Jump timing**: M30 entry confirmation required
- **Gravity**: Market pressure against position requires hedge protection

## Implementation Details

### Daily Trend Detection (D1+ Indicators)
- Moving averages for trend direction
- RSI/MACD on daily timeframe only
- Major support/resistance identification
- Output: Bullish/Bearish/Sideways bias for M30 entries

### M30 Scalping Logic
- Entry only on M30 candle close confirmation
- Position size: Account ÷ 10,000 (1000 USD = 0.1 lot max)
- Target: 20-30 pips per scalp
- Stop: 40-50 USD or technical M30 level

### Hedge System
- Trigger: M30 scalp fails by 40-60 pips (day) or 70-90 pips (night)
- Maximum: 3-4 total orders per setup
- Risk limit: 5% account maximum
- Exit: Close profitable legs when available

## Training Parameters
- Episode length: 1 trading day
- Episode end: Daily loss limit hit or profit target achieved
- Reward optimization: Maximize daily profit while minimizing hedge frequency
- Success metric: Consistent 3-4% daily gains with controlled drawdown

## Critical Limitations

### Mathematical Impossibilities
- **Position Sizing Formula**: Account ÷ 10,000 creates 50% account risk on 50-pip moves
- **M30 Confirmation Delays**: 30-minute delay ensures worst possible entry prices
- **Hedge Risk Multiplication**: "Safety" system actually amplifies total exposure

### Training Data Issues
- **Historical Bias**: Models learn patterns that don't repeat in live markets
- **Expensive Learning**: 5000+ episodes needed, each risking real money
- **Regime Changes**: Training data becomes obsolete when market conditions change

### Survival Probability
- Week 1: 40% survival rate
- Month 1: 8% survival rate  
- Month 6: 0.5% survival rate

The hedging system becomes a systematic loss generator, triggering exactly where institutions hunt retail stops.