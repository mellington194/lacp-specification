# LACP Talent: Deep Value Screener (Financial Analysis Pipeline)

## 1. Overview

**Deep Value Screener** (DVS) is an AI-powered financial analysis swarm that screens equities through fundamental analysis, risk assessment, sentiment scanning, and historical backtesting. It operates 4 specialised agents that combine quantitative metrics with qualitative market signals to produce actionable investment insights.

The swarm implements a **multi-persona analysis model** — generating Bull, Bear, Value, and Growth perspectives for each screened security — ensuring that no single analytical bias dominates the output. Agents operate against a FastAPI backend with PostgreSQL 16 (pgvector) for semantic search and DuckDB for OLAP analytics.

## 2. Swarm Architecture

DVS operates a **3-stage pipeline** with 4 specialised agents. Stage 1 executes fundamental and sentiment analysis in parallel, feeding into sequential risk synthesis and backtesting.

```
    +----------------------------+----------------------------+
    |                                                         |
    v                                                         v
Stage 1a: ANALYST (Analyst)                Stage 1b: SENTIMENT_SCANNER (Explorer)
    |   40+ financial metrics                  |   NLP analysis, earnings call parsing,
    |   DCF models, ratio analysis             |   Reddit/Twitter signal extraction
    |   Sector comparisons                     |
    +----------------------------+----------------------------+
                                 |
                                 v
Stage 2: RISK_ASSESSOR (Security)
    |   Volatility analysis, drawdown simulation,
    |   correlation matrices, regime detection
    |
    v
Stage 3: BACKTESTER (Tester)
        Strategy backtesting, Monte Carlo simulation,
        walk-forward analysis
```

**Design Rationale:**
*   Stage 1 branches are independent — fundamental analysis and sentiment scanning have no data dependencies. Parallel execution reduces screening time by ~35%.
*   The Risk Assessor synthesises outputs from both upstream agents, weighting quantitative metrics against qualitative sentiment signals.
*   The Backtester runs last because it needs the complete risk-adjusted thesis to construct a testable strategy. Walk-forward validation ensures the strategy is not overfit to historical data.

## 3. Agent Roles

| Agent | LACP Role | Archetype | Model | Trust Level | Function |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Analyst** | `ANALYST` | Quant | `claude-sonnet-4-5` | `INTERNAL` | Fundamental analysis: 40+ financial metrics, DCF models, ratio analysis, sector comparisons |
| **Risk Assessor** | `SECURITY` | Guardian | `claude-sonnet-4-5` | `INTERNAL` | Risk scoring: volatility analysis, drawdown simulation, correlation matrices, regime detection |
| **Sentiment Scanner** | `EXPLORER` | Scout | `gemini-2.0-flash` | `EXTERNAL` | News and social sentiment: NLP analysis, earnings call parsing, Reddit/Twitter signal extraction |
| **Backtester** | `TESTER` | Validator | `claude-sonnet-4-5` | `INTERNAL` | Historical validation: strategy backtesting, Monte Carlo simulation, walk-forward analysis |

**Model Selection Notes:**
*   The Analyst, Risk Assessor, and Backtester use `claude-sonnet-4-5` for structured reasoning — financial modelling and risk synthesis require precise numerical analysis.
*   The Sentiment Scanner uses `gemini-2.0-flash` for cost-efficient processing of high-volume social and news data. Speed matters more than depth for signal extraction.

## 4. LACP Interface

### 4.1 Role Mapping

| Domain Role | LACP Role | Justification |
| :--- | :--- | :--- |
| `ANALYST` | `ANALYST` | Analyses financial data to produce structured investment metrics |
| `RISK_ASSESSOR` | `SECURITY` | Secures the portfolio against risk — identifies threats and vulnerabilities |
| `SENTIMENT_SCANNER` | `EXPLORER` | Explores unstructured data sources for market signals |
| `BACKTESTER` | `TESTER` | Tests investment theses against historical data |

### 4.2 Trust Level

All agents operate at `INTERNAL` trust except the Sentiment Scanner, which operates at `EXTERNAL` trust due to its ingestion of third-party social media and news feeds. External data is sanitised and confidence-scored before being passed to downstream agents.

### 4.3 Context Object

```json
{
  "mission_id": "dvs-screen-{ticker}-{timestamp}",
  "objective": "Screen equity for investment potential",
  "payload": {
    "ticker": "LVMH.PA",
    "perspectives": ["bull", "bear", "value", "growth"],
    "timeHorizon": "12m",
    "benchmarks": ["CAC40", "SP500"],
    "constraints": {
      "maxDrawdown": 0.20,
      "minSharpe": 1.0
    },
    "formicContext": {
      "memories": [],
      "genePoolPatterns": [],
      "previousScreenings": []
    }
  }
}
```

*   **`ticker`** — Exchange-qualified ticker symbol.
*   **`perspectives[]`** — Multi-persona analysis viewpoints to generate.
*   **`timeHorizon`** — Investment horizon for DCF and backtesting.
*   **`benchmarks[]`** — Indices for relative performance comparison.
*   **`constraints`** — Risk parameters: maximum drawdown tolerance and minimum Sharpe ratio.

### 4.4 Feature Flag

```
USE_DVS_SWARM=true
```

*   **`true`** — Activates the full 4-agent pipeline with multi-persona analysis.
*   **`false`** — Falls back to single-model screening with default parameters.
*   **Auto-fallback:** If market data feeds are unavailable, the system degrades gracefully — the Analyst operates on cached data and the Sentiment Scanner is skipped.

## 5. Tribunal Integration

DVS defines **4 domain-specific conditions** that trigger a Tribunal review. When triggered, the pipeline pauses and a Tribunal is convened using the standard Commit-Reveal flow (see `../consensus/tribunal.spec.md`).

### 5.1 Trigger Conditions

| # | Condition | Trigger Rule | Jury Composition |
| :--- | :--- | :--- | :--- |
| 1 | **Conflicting signals** | ANALYST bullish but SENTIMENT_SCANNER bearish (divergence > 0.4) | `ANALYST` + `SENTIMENT_SCANNER` + `RISK_ASSESSOR` |
| 2 | **High-risk position** | RISK_ASSESSOR scores position > 0.8 risk | `RISK_ASSESSOR` + `ANALYST` + `BACKTESTER` |
| 3 | **Backtest failure** | Strategy fails walk-forward validation (out-of-sample Sharpe < 0.5) | `BACKTESTER` + `RISK_ASSESSOR` + `ANALYST` |
| 4 | **Data quality** | Missing or stale market data detected (> 24h latency) | Full panel (all 4 agents) |

### 5.2 Tribunal Mechanics

*   **Voting:** Personality-weighted blind voting with sycophancy detection (see `../consensus/sycophancy.spec.md`).
*   **Personality Weighting:** Agents with higher Conscientiousness and lower Agreeableness receive higher effective weight. The Risk Assessor's naturally low Agreeableness gives it outsized influence in risk-related tribunals.
*   **Safety Default:** If the Tribunal cannot reach consensus, the screening is flagged as "inconclusive" — no buy/sell signal is emitted.
*   **Escalation:** Screenings that fail Tribunal twice are escalated to human review with all agent reasoning traces and dissenting opinions attached.

## 6. Personality System (AAPS)

Each DVS agent has an **OCEAN personality profile** managed by the Agent Adaptive Personality System (AAPS). Refer to `../consensus/personality.spec.md` for the full specification.

### 6.1 Default Archetypes

Temperature formula: `temp = clamp(O * 0.6 + E * 0.2 - C * 0.15 - N * 0.1, 0.0, 1.0)`

| Agent | Archetype | O | C | E | A | N | Derived Temp |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Analyst | Quant | 0.55 | 0.85 | 0.40 | 0.45 | 0.50 | ~0.23 |
| Risk Assessor | Guardian | 0.30 | 0.90 | 0.30 | 0.20 | 0.80 | ~0.03 |
| Sentiment Scanner | Scout | 0.80 | 0.60 | 0.75 | 0.55 | 0.35 | ~0.51 |
| Backtester | Validator | 0.25 | 0.95 | 0.25 | 0.30 | 0.65 | ~0.00 |

**Calibration Note:** The Risk Assessor and Backtester have near-zero derived temperatures, reflecting their conservative, deterministic nature. The Risk Assessor's high Neuroticism (0.80) and low Agreeableness (0.20) make it the "paranoid bear" of the swarm — it actively resists consensus and flags edge-case risks. The Sentiment Scanner has the highest temperature (~0.51), enabling it to surface weak signals from noisy social data.

### 6.2 Personality Evolution

Personality profiles evolve based on three feedback signals:

1.  **Market Outcomes** — When a screened position performs significantly better or worse than predicted, all agents are nudged. Consistently wrong Risk Assessors have Neuroticism increased; consistently wrong Analysts have Conscientiousness increased.
2.  **Tribunal Outcomes** — Agents whose votes align with the final verdict receive a small Conscientiousness boost.
3.  **Prediction Accuracy** — Tracked over rolling 90-day windows. Agents with declining accuracy have Openness reduced (making them more conservative) while consistently accurate agents have Openness increased (enabling more exploratory analysis).

## 7. Tech Stack

| Component | Technology | Purpose |
| :--- | :--- | :--- |
| Backend | FastAPI | API layer and agent orchestration |
| Primary DB | PostgreSQL 16 + pgvector | Structured financial data and semantic search |
| Analytics | DuckDB | OLAP queries for backtesting and ratio analysis |
| Market Data | Real-time feeds | Live pricing, volume, and order book data |
| Embeddings | pgvector | Semantic similarity for comparable company analysis |

## 8. Cross-Swarm Interactions

| Pattern Flow | Source | Destination | Example |
| :--- | :--- | :--- | :--- |
| Pricing/resale data | Provenance | **DVS** | Luxury resale market pricing enriches sector analysis for LVMH, Hermes, etc. |
| Financial patterns | DVS | **Gene Pool** | Successful screening patterns and risk models shared as reusable templates |
| Trend detection | DVS | **Knowledge Archive** | Market regime detection methods and sentiment analysis techniques |
| Risk models | DVS | **SafetyOS** | Financial risk scoring patterns inform broader safety heuristics |
| Market context | DVS | **Provenance** | Market trend data informs Appraiser pricing for luxury goods |

**Propagation Mechanism:** All cross-swarm interactions flow through the Gene Pool's async push/pull protocol. No direct swarm-to-swarm communication occurs — the Gene Pool acts as the sole intermediary, ensuring workspace isolation and audit traceability.
