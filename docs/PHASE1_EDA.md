# Phase 1: Exploratory Data Analysis

## Phase Overview

**Duration:** 3-4 weeks
**Status:** 🔵 Not started
**Branch:** `phase-1/eda`

### Objectives

1. Establish reliable data pipelines for market data
2. Prototype VPIN calculation and validate against known events
3. Analyze correlation dynamics during market stress
4. Evaluate sentiment data sources for FinBERT integration
5. Collect historical model performance data for trustless scoring validation

---

## Task 1.1: Market Data Pipeline Setup

**Task ID:** `EDA-001`
**Estimated Time:** 3-4 days
**Dependencies:** None

### Description

Set up data ingestion pipeline for cryptocurrency and traditional market data. Must support real-time streaming and historical backfill.

### Acceptance Criteria

- [ ] Connect to 2+ crypto exchanges (Binance, Coinbase)
- [ ] Inget OHLCV data at 1-minute granularity
- [ ] Store in time-series database (TimescaleDB/InfluxDB)
- [ ] Data completeness validation > 99%
- [ ] Backfill capability for 2+ years of historical data

## AI Prompt

```Claude CLI
You are a senior data engineer building a market data pipeline for a quantitative trading platform.

CONTEXT:
- Target assets: Top 50 cryptocurrencies by market cap
- Data sources: Binance API, Coinbase Pro API
- Granularity: 1-minute OHLCV (Open, High, Low, Close, Volume)
- Storage: TimescaleDB for time-series optimization
- Requirements: Real-time streaming + historical backfill

TASK:
1. Design the data pipeline architecture (diagram + explanation)
2. Implement Python data ingestion service with:
  - WebSocket connections for real-time data
  - REST API fallback for gap filling
  - Automatic reconnection logic
  - Data validation and deduplication
3. Create TimescaleDB schema optimized for:
  - Fast time-range queries
  - Efficient aggregation (1min → 5min → 1hour)
  - Compression for historical data
4. Implement backfill script for historical data
5. Add monitorig for data completeness

OUTPUT FORMAT:
- Architecture diagram (ASCII or description)
- Python code with type hints and docstrings
- SQL schema definitions
- Docker Compose for local development
- Unit tests for critical paths

CONSTRAINTS:
- Handle rate limits gracefully
- Support multiple exchange formats
- Log all data quality issues
- Must be idempotent (safe to re-run)
- Suggested improvements for future iterations
```

---

## Task 1.2: VPIN Calculation Engine Prototype

**Task ID:** `EDA-002`
**Estimated Time:** 4-5 days
**Dependencies:** `EDA-001`

### Description

Implement Volume-Synchronized Probability of Informed Trading (VPIN) calculation. VPIN measures order flow toxicity and predicts adverse selection risk.

### Acceptance Criteria

- [ ] VPIN calculation matches academic specifications
- [ ] Calculation latency < 100ms per update
- [ ] Validated against known flash crash events
- [ ] Configurable bucket size and lookback window
- [ ] Real-time streaming capability

### AI Prompt

```Claude CLI
You are a quantitative researcher implementing VPIN (Volume-Synchronized Probability of Informed Trading) for a trading plateform.

CONTEXT:
VPIN is a measure of order flow toxicity that predicts maker adverse selection risk. High VPIN (>0.8) indicates informed traders are active.

VPIN CALCULATION:
1. Divide trading volume into equal-sized "buckets" (e.g., 1/50 th of daily volume)
2. Classify each trade as buy or sell (using tick rule or Lee-Ready algorithm)
3. Calculate Order Imabalnce (OI) = |Buy Volume - Sell Volume| / Total Volume per bucket
4. VPIN = Moving average of OI over N buckets (typically 50)

TASK:
1. Implement VPIN calculator class with:
  - Configurable bucket size (volume-based, not time-based)
  - Trade classification using tick rule
  - Rolling window calculation
  - Real-time update capability
2. Create validation against historical events:
  - May 6, 2019 Flash Crash (VPIN should spike before crash)
  - March 2020 COVID crash
  - Any crypto flash crashes (May 2021, etc.)
3. Impelement streaming interface for real-time calculation
4. Add visualization for VPIN over time

OUTPUT FORMAT:
- Python class with full type annotations
- Mathematical documentation of the algorithm
- Validation report with charts
- Performance benchmarks
- Unit tests with edge cases

CONSTRAINTS:
- Must handle gaps n data gracefully
- Calculation must be deterministic (same input → same output)
- Memory-efficient for long-running processes

After completing this task, update MILESTONES.md with:
  - Task completion status
  - Validation results against known events
  - Performance metrics achieved
  - Suggested calibration parameters
```

---

## Task 1.3: Flash Crash Correlation Analysis

**Task ID:** `EDA-003`
**Estimated Time:** 3-4 days
**Dependencies:** `EDA-001`

### Description

Analyze how asset correlations behave during market stress events. This informs CVaR implementation when diversification fails.

### Acceptance Criteria

- [ ] Identify 5+ historical flash crash events
- [ ] Calculate rolling correlation matrices (1h, 4h, 24h windows)
- [ ] Document correlation regime changes during stress
- [ ] Quantify "correlation breakdown" threshold
- [ ] Provide recommendation for CVaR triggers

### AI Prompt

```Claude CLI
You are a quantitative risk analyst studying correlation dynamics during market stress events.

CONTEXT:
During flash crashes, asset correlations approach 1.0, causing diversification to fail. Traditional Mean-Variance Optimization (Markowitz) assumes stable correlations, which breaks down in crisis.

Our portfolio engine needs to:
1. Detect when correlation spike (regime change)
2. Switch from Markowitz to CVaR optimization.
3. Define threshold triggers for this switch.

TASK:
1. Identify and document 5+ historical flash crash events:
  - Traditional markets: 2010 Flash Crash, 2015 ETF Flash Crash, COVID March 2020.
  - Crypto: May 2021, Terra/Luna collapse, FTX collapse
2. For each event, calculate:
  - Rolling correlation matrix (1h, 4h, 24h lookback)
  - Average pairwize correlation over time
  - Time from correlation spike to price crash
  - Recovery time for correlations to normalize
3. Statistical Analysis:
  - What correlation level indicates "crisis mode"?
  - How fast do correlations spike? (warning time)
  - False positive rate for correlation-based triggers
4. Recommendations:
  - Threshold for switching to CVaR
  - Lookback window for correlation monitoring
  - Hysteresis to prevent oscillation

OUTPUT FORMAT:
- Jupyter notebook with analysis.
- Summary report with charts
- Reommended parameters with justification.
- Python utility functions for correlation monitoring


CONSTRAINTS:
- Use consistent methodology across all events.
- Account for different market hours (crypto 24/7 vs traditional)
- Consider data quality issues during stress (gaps, delays)

After completing this task, update MILESTONES.md with:
- Task completion status
- Recommended correlation threshold
- Key findings about correlation dymanics
- Data quality issues encountered
```

---

## Task 1.4: Sentiment Data Source Evaluation

**Task ID:** `EDA-004`
**Estimated Time:** 3-4 days
**Dependencies:** None

### Description

Evaluate sentiment data sources for integration with FinBERT sentiment module. Compare latency, coverage, and signal quality.

### Acceptance Criteria

- [ ] Evaluate +3 sentiment data sources.
- [ ] Measure latency from event to data availability
- [ ] Assess coverage of macro news events
- [ ] Test FinBERT inference on sample data
- [ ] Recommend primary and backup source

### AI Prompt:

```Claude CLI
You are a data scientist evaluating sentiment data sources for a trading platform's NLP pipeline.

CONTEXT:
Our sentiment module uses FinBERT for scoring breaking macro news. The architecture is:
- Async "Sidecar" service runs FimBERT
- Results cached for retrieval by execution engine
- Need reliable, low-latency news feeds

REQUIREMENTS:
- Macro news coverage (Fed, ECB, economic releases)
- Crypto-specific news (exchange issues, regulatory)
- Latency: Event → Data availabality < 5 seconds ideal
- Volume: Handle 100+ articles/hour during high activity

TASK:
1. Evaluate these data sources:
  - News API: NewsAPI, Bloomberg, Reuters, Alpha Vantage
  - Social: Twitter/X API, Reddit (r/cryptocurrency, r/wallstreetbets)
  - Crypto-specific: CryptoPanic, LunarCrush, Santiment
2. For each source, measure:
  - Latency (event timestamp vs API availability)
  - Coverage (% of major events captured)
  - Data quality (noise ratio, relevance)
  - API limits and pricing
  - Historical data availability
3. Test FinBERT on sample data:
  - Donwload/setup FinBERT model
  - Run inference on 100 sample headlines
  - Measure inference latency
  - Evaluate sentiment accuracy on labeled samples
4. Architecture recommendations:
  - Primary and backup data sources
  - Deduplication strategy
  - Priority queue design for breaking news

OUTPUT FORMAT:
- Comparison matrix of data sources
- FinBERT performance benchmarks
- Sample code for each API integration
- Architecture recommendation document

CONSTRAINTS:
- Stay within free tier for initial evaluation
- Consider GDPR/data retention requirements
- Account for API reliability during market stress

After completing this task, update MILESTONES.md with:
- Task completion status
- Recommended data sources with reasoning
- FinBERT latency benchmarks
- Cost estimates for production scale
```

---

## Task 1.5: Historical Model Performance Dataset

**Task ID:** `EDA-005`
**Estimated Time:** 4-5 days
**Dependencies:** `EDA-001`

### Description
Create synthetic and real historical model performance data for validating the trustless scoring engine and Multi-Armed Bandit allocator.

### Acceptance Criteria

- [ ] Generate 50+ synthetic model performance histories
- [ ] Include various archetypes (consistent, volatile, regime-dependent)
- [ ] Create "Gambler" models that occasionally spike
- [ ] Document ground truth for validation
- [ ] Format compatible with trustless scoring engine

### AI Prompt

```Claude CLI
You are a quantitative researcher creating test datasets for a model evaluation system.

CONTEXT:
Our plateform needs to:
1. Score models using Beta-Bernoully with exponential decay
2. Allocate capital using Multi-Armed BAndit (UCB)
3. Detect and penalize "Gambler" models (high variance, occasional spikes)
4. Use Sortiono ratio for RL reward function

We need realistic test data to validate these systems before going live.

TASK:
1. Define model archetype (10+ types):
  - Consistent Performer: 55% win rate, low variance
  - Volatile Winner: 52% win rate, high variance
  - Regime-Dependent: Good in trends, bad in ranges
  - Gambler: 40% win rate but occasional 10x wins
  - Decaying Alpha: Started strong, degrading over time
  - Session-Specific: Good during US hours, bad during Asia gap
  - Mean Reverter: Profits in calm markets, loses in trends
  - Momentum Follower: Opposite of mean reverter
  - Random Walk: No edge, pure noise
  - Broken model: Was good, then regime shift broke it
2. Generate synthetic performance data:
  - 50 models, 6 months of daily returns each
  - Include timestamps, PmL, win/loss, position sizes
  - Add realistic noise and correlation
3. Create ground truth labels:
  - True Sharpe ratio for each model
  - True Sortino ratio
  - Regime labels (trending, ranging, crisis)
  - "Should allocate" recommendation
  - Parquet for efficient storage

OUTPUT Format:
- Python script for data generation (reproducible with seed)
- Generated datasets in multiple formats
- Documentation of each archetype
- Visualization of model performance distributions
- Ground truth file for validation

CONSTRAINTS:
- Realistic return distributions (fat tails)
- Correlated drawdowns during stress periods
- Some models should have regime breaks mid-history

After completing this task, update MILESTONES.md with:
- Task completion status
- Number of models generate per archetype
- Statistic properties of generated data
- Recommended test scenarios for each system component
```

---

## Task 1.6: Data Quality Framework

**Task ID:** `EDA-006`
**Estimated Time:** 2-3 days
**Dependencies:** `EDA-001`, `EDA-004`

### Description

Establish data quality monitoring and alerting framework for all data pipelines.

### Acceptance Criteria

- [ ] Define data quality metrics (completeness, latency, accuracy)
- [ ] Implement automated quality checks
- [ ] Create alerting for quality degradation
- [ ] Dashboard for data quality monitoring
- [ ] Runbook for common data issues

### AI Prompt

```Claude CLI
You are a data quality engineer establishing monitoring for a trading platform's data infrastructure.

CONTEXT:
Data quality is critical for trading systems. Bad data leads to:
- Incorrect VPIN calculations → wrong execution decisions
- Stale sentiment → missed opportunities
- Gaps in price data → backtesting errors

DATA SOURCES TO MONITOR:
1. Market data pipeline (OHLCV from exchanges)
2. Sentiment feeds (news API, social)
3. Model signals (user-submitted predictions)

TASK:
1. Define quality metrics for each data source:
  - Completeness: % of expected data points received
  - Latency: Time from source to storage
  - Accuracy: Validation against secondary sources
  - Freshness: Age of most recent data point
  - Schema compliance: Format validation
2. Implement automated checks:
  - Real-time validation on ingestion
  - Periodic batch reconciliation
  - Cross-source consistency checks
3. Create alerting rules:
  - Thresholds for each metric
  - Escalation paths
  - Alert fatigue prevention (aggregation, cooldown)
4. Build monitoring dashboard:
  - Real-time quality scores
  - Historical trends
  - Drill-down by source/asset
5. Document runbooks:
  - Common failure modes
  - Troubleshooting steps
  - Recovery procedures

OUTPUT FORMAT:
- Python quality check library
- Prometheus metrics library
- Grafana dashboard JSON
- AlertManager configuration
- Runbook documentation (Markdown)

CONSTRAINTS:
- Checks must not impact ingestion latency significantly
- Support both streaming and batch validation
- Historical quality data retention (+30 days)

After completing this task, update MILESTONES.md with:
- Task completion status
- Quality metrics baseline measurements
- Alert thersholds configured
- Known data quality issues documented
```

---

## Phase 1 Completion Checklist

| Task ID | Task Name | Status | Completed Date |
|---------|-----------|--------|----------------|
| EDA-001 | Market Data Pipeline | ⬜ | - |
| EDA-002 | VPIN Calculation Engine | ⬜ | - |
| EDA-003 | Flash Crash Correlation Analysis | ⬜ | - |
| EDA-004 | Sentiment Data Source Evaluation | ⬜ | - |
| EDA-005 | Historical Model Performance Dataset | ⬜ | - |
| EDA-006 | Data Quality Framekwork | ⬜ | - |

## Phase 1 Exit Criteria

Before proceeding to Phase 2, ensure:

1. ✅ All data pipelines operational with >99% completeness
2. ✅ VPIN validated against at least 2 historical events
3. ✅ Correlation thesholds defined for CVaR triggers
4. ✅ Sentiment data source selected and tested
5. ✅ Test datasets ready for module validation
6. ✅ Data quality monitoring active

## Related Documents

- [Milestone](../MILESTONES.md)
- [READING LIST](../resources/PHASE_1_EDA.md)
