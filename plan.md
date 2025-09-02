# Forex DQN Trading System Refactor Plan

## Project Overview
Transform the Flappy Bird DQN codebase into a production-ready forex trading system using pure price action analysis, daily range statistics, and psychological levels.

## System Architecture

### Core Components Refactoring

#### 1. Environment Replacement (`src/flappy_bird.py` → `src/forex_environment.py`)
- Replace pygame game mechanics with MT5 market data feed
- Implement tick-by-tick price streaming with configurable timeframes
- Add session-aware trading (London/NY overlap detection)
- Integrate daily range calculation and pivot level identification

#### 2. State Space Design (`DeepQNetwork` input layer modification)
**Replace 84x84 game pixels with 12-dimensional market state vector:**
- Current price position within daily range (0-1 normalized)
- Distance to nearest round number (00, 50 levels)
- Time within trading session (London: 0-1, NY: 0-1, Overlap: 0-1)
- Daily range completion percentage
- Volume profile position (high/medium/low volume zone)
- Previous day's high/low breach status
- Weekly pivot relationships (above/below R1, S1, etc.)
- Account risk utilization (0-1 normalized to 2% max)

#### 3. Action Space Redefinition
**5 discrete actions replacing Flappy Bird's binary choice:**
- **0**: Wait/Hold current position
- **1**: Enter Long (buy signal)
- **2**: Enter Short (sell signal)  
- **3**: Scale In (add to winning position)
- **4**: Close Position (take profit/stop loss)

#### 4. Reward Function (`next_frame()` reward logic)
**Pure performance-based rewards:**
- **+100**: Hit daily profit target (1-2% account growth)
- **+50**: Close position at round number reversal
- **+25**: Successful scale-in execution
- **-75**: Stop loss triggered
- **-100**: Exceed 2% account risk limit
- **-25**: Trading outside optimal sessions
- **+10**: Each 10-pip unrealized profit
- **-5**: Each 10-pip unrealized loss

## Technical Implementation

### Data Pipeline Architecture

#### Market Data Handler
```
src/market_data.py
├── MT5Connection: Live price feeds
├── SessionManager: London/NY session detection
├── RangeCalculator: Daily/weekly range statistics
├── LevelIdentifier: Round numbers, pivots, S/R zones
└── VolumeProfiler: Tick volume analysis
```

#### Feature Engineering Pipeline
```
src/features.py
├── PriceNormalizer: Scale prices to 0-1 ranges
├── SessionFeatures: Time-based market context
├── RangeFeatures: Daily range positioning
├── LevelProximity: Distance to psychological levels
└── RiskFeatures: Account exposure calculations
```

#### Environment Integration
```
src/forex_environment.py
├── step(): Execute trade action, return new state/reward
├── reset(): New trading day initialization
├── render(): Optional price chart visualization
├── _calculate_reward(): Performance-based reward logic
└── _get_observation(): 12D state vector construction
```

### Model Architecture Updates

#### Network Modifications (`src/deep_q_network.py`)
- **Input Layer**: Change from (4, 84, 84) to (12,) vector
- **Hidden Layers**: 3 fully connected layers [512, 256, 128]
- **Output Layer**: 5 actions instead of 2
- **Activation**: ReLU hidden, linear output for Q-values
- **Regularization**: Dropout layers for overfitting prevention

#### Training Loop Enhancements (`train.py`)
- **Episode Structure**: 24-hour trading days
- **Memory Management**: Circular buffer for experience replay
- **Target Network**: Separate target network for stable learning
- **Exploration Strategy**: ε-greedy with session-aware decay
- **Performance Tracking**: Daily P&L, win rate, max drawdown

### Risk Management System

#### Position Sizing Logic
- **Base Size**: 0.01 lots per $1000 account balance
- **Risk Limit**: Maximum 2% account per trade
- **Scaling Rules**: Add to position only at 20+ pip profit
- **Session Limits**: Reduced size during Asian session

#### Safety Mechanisms
- **Daily Drawdown Limit**: 5% account, force episode termination
- **News Event Filter**: Halt trading 30 minutes before/after high-impact news
- **Weekend Gap Protection**: Close all positions before market close
- **Connection Loss Handler**: Emergency position closure on MT5 disconnect

## Training Strategy

### Environment Setup
1. **Historical Data**: 2 years M1 tick data for major pairs (EURUSD, GBPUSD, USDJPY)
2. **Validation Split**: Train on 18 months, validate on 6 months
3. **Walk-Forward Testing**: Retrain every 3 months on rolling window

### Hyperparameter Optimization
- **Learning Rate**: 1e-4 (reduced from original 1e-6)
- **Batch Size**: 64 experiences per update
- **Memory Size**: 100,000 recent experiences
- **Epsilon Decay**: Linear from 0.9 to 0.05 over 500,000 steps
- **Gamma**: 0.95 (shorter-term focus than original 0.99)

### Training Phases
1. **Phase 1 (100k steps)**: Learn basic buy/sell logic
2. **Phase 2 (300k steps)**: Develop session timing awareness  
3. **Phase 3 (500k steps)**: Master position scaling and exit timing
4. **Phase 4 (200k steps)**: Fine-tune risk management

## Production Deployment

### Live Trading Pipeline
```
production/
├── live_trader.py: Main trading execution loop
├── risk_monitor.py: Real-time risk assessment
├── performance_tracker.py: P&L and metrics logging
├── alert_system.py: SMS/email notifications
└── backup_systems.py: Failsafe mechanisms
```

### Monitoring Dashboard
- **Real-time P&L**: Current session performance
- **Risk Metrics**: Current exposure, daily drawdown
- **Model Confidence**: Q-value distributions per action
- **Market Conditions**: Session detection, range completion
- **System Health**: MT5 connection, model prediction latency

## Directory Structure Refactoring

```
forex-dqn-trader/
├── README.md
├── requirements.txt
├── config/
│   ├── trading_params.yaml
│   └── model_config.yaml
├── src/
│   ├── forex_environment.py
│   ├── deep_q_network.py
│   ├── market_data.py
│   ├── features.py
│   ├── risk_manager.py
│   └── utils.py
├── training/
│   ├── train_dqn.py
│   ├── backtest.py
│   └── optimize_hyperparams.py
├── production/
│   ├── live_trader.py
│   ├── risk_monitor.py
│   └── performance_tracker.py
├── data/
│   ├── historical/
│   └── models/
├── tests/
│   ├── test_environment.py
│   ├── test_features.py
│   └── test_risk_manager.py
└── logs/
    ├── training/
    └── trading/
```

## Migration Checklist

### Phase 1: Core Infrastructure
- [ ] Install MT5 Python integration
- [ ] Set up market data pipeline
- [ ] Implement basic forex environment
- [ ] Test state space generation
- [ ] Validate reward function logic

### Phase 2: Model Adaptation  
- [ ] Modify DQN architecture for vector input
- [ ] Update training loop for forex episodes
- [ ] Implement target network updates
- [ ] Add experience replay improvements
- [ ] Test model convergence on historical data

### Phase 3: Risk & Production Systems
- [ ] Build comprehensive risk manager
- [ ] Implement live trading interface
- [ ] Create monitoring dashboard
- [ ] Set up alerting system
- [ ] Deploy backup/failsafe mechanisms

### Phase 4: Testing & Validation
- [ ] Extensive backtesting on unseen data
- [ ] Paper trading validation (1 month minimum)
- [ ] Performance benchmark against buy-and-hold
- [ ] Stress testing with various market conditions
- [ ] Live trading with minimum position sizes

## Success Metrics

### Training Objectives
- **Convergence**: Stable Q-values after 1M training steps
- **Sample Efficiency**: Positive returns within 200k steps
- **Generalization**: Consistent performance across different currency pairs

### Live Trading Targets
- **Monthly Return**: 8-15% with maximum 5% drawdown
- **Win Rate**: 60%+ on executed trades
- **Sharpe Ratio**: 2.0+ risk-adjusted returns
- **Maximum Consecutive Losses**: 5 trades before system pause

## Risk Warnings

### Model Limitations
- **Regime Changes**: Model requires retraining when market conditions shift
- **Black Swan Events**: No protection against unprecedented market movements  
- **Slippage Impact**: Live execution differs from backtested performance
- **Latency Sensitivity**: Network delays can invalidate model predictions

### Capital Requirements
- **Minimum Account**: $10,000 for meaningful position sizing
- **Reserve Capital**: 50% additional funds for drawdown periods
- **Technology Costs**: VPS hosting, data feeds, monitoring tools
- **Learning Period**: 6-12 months of development before profitable deployment