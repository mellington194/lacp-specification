# LACP Consensus: Adaptive Agent Personality System (AAPS)

## 1. Overview
Personality profiles replace hardcoded agent configurations with parametric behaviour derived from the **OCEAN** (Big Five) personality dimensions. Rather than defining each agent's behaviour through static config values, AAPS expresses agent tendencies as a five-dimensional personality vector. Concrete parameters (temperature, confidence thresholds, vote weight) are then *derived* from this vector, enabling emergent behavioural diversity and gradual evolution over time.

**Design Goals:**
*   Eliminate brittle per-agent config files in favour of a single parametric model.
*   Allow agent behaviour to evolve in response to feedback without manual tuning.
*   Maintain deterministic derivation — the same OCEAN scores always produce the same parameters.
*   Preserve backwards compatibility — default OCEAN scores produce behaviour equivalent to legacy static configs.

## 2. OCEAN Dimensions
Each agent carries a five-dimensional personality vector. All dimensions use a continuous `0.0`–`1.0` scale, **bounded to `[0.05, 0.95]`** to prevent degenerate behaviour at the extremes.

| Dimension | Code | Low (→ 0.0) | High (→ 1.0) |
| :--- | :--- | :--- | :--- |
| **Openness** | `O` | Conservative, routine-following | Exploratory, creative |
| **Conscientiousness** | `C` | Flexible, fast, loose checks | Meticulous, thorough, slow |
| **Extraversion** | `E` | Reserved, minimal output | Verbose, proactive communication |
| **Agreeableness** | `A` | Contrarian, independent | Deferential, consensus-seeking |
| **Neuroticism** | `N` | Calm, risk-tolerant | Anxious, cautious, escalation-prone |

**Boundary Enforcement:**

```
clamp(value) = max(0.05, min(0.95, value))
```

Any mutation (evolution event, manual override) passes through `clamp()` before persistence.

## 3. Derived Behaviour Parameters
OCEAN scores are mapped to concrete runtime parameters via deterministic formulae. All derivations are pure functions — no hidden state.

### 3.1 `temperature`
How creative or conservative the agent's LLM sampling is.

```
temperature = clamp(O * 0.6 + E * 0.2 - C * 0.15 - N * 0.1, 0.0, 1.0)
```

*   High Openness and Extraversion push creative output.
*   High Conscientiousness and Neuroticism pull towards deterministic output.

### 3.2 `topP`
Nucleus sampling diversity.

```
topP = clamp(0.5 + O * 0.3 + E * 0.15, 0.5, 1.0)
```

*   Derived primarily from Openness with an Extraversion boost.
*   Floor of 0.5 ensures minimum sampling diversity.

### 3.3 `confidenceThreshold`
Minimum confidence to accept own output without escalation.

```
confidenceThreshold = clamp(0.4 + C * 0.35 + N * 0.15, 0.4, 0.95)
```

*   Conscientious agents demand higher self-confidence before committing.
*   Neurotic agents also set the bar higher (anxiety about mistakes).

### 3.4 `escalationThreshold`
Confidence floor below which the agent triggers a Tribunal.

```
escalationThreshold = clamp(0.7 - N * 0.3 + A * 0.1, 0.3, 0.8)
```

*   High Neuroticism lowers the threshold (more frequent escalation).
*   High Agreeableness raises it slightly (prefers to defer rather than force a Tribunal).

### 3.5 `tribunalVoteWeight`
Influence weight when casting votes in Tribunal consensus.

```
tribunalVoteWeight = clamp(0.5 + C × 0.6 − A × 0.15, 0.5, 1.2)
```

*   Range: `0.5`–`1.2`.
*   Conscientiousness is the primary positive driver — meticulous agents carry more weight.
*   Agreeableness acts as a negative modifier — high Agreeableness agents are more susceptible to sycophancy and should carry slightly less tribunal weight, reducing their influence on consensus outcomes.

### 3.6 Temperature Calibration Bands
The `temperature` derivation (Section 3.1) produces values that cluster into four operational bands. Implementations SHOULD use these bands to validate archetype configurations and flag anomalies.

| Band | Range | Representative Archetypes | Use Case |
| :--- | :--- | :--- | :--- |
| Conservative | 0.15–0.25 | Sentinel, Replete, Inspector, Ergate | Precision tasks, auditing, safety |
| Moderate | 0.25–0.45 | Mason, Pheromone, Forager, Carrier, Steward | Planning, coordination, analysis |
| Balanced | 0.45–0.55 | Nurse, Antenna, Scholar | Documentation, coaching, research |
| Creative | 0.55–0.70 | Morph, Scout, Gyne, Rogue | Design, exploration, strategy |

**Note:** Temperatures above 0.70 are permitted but indicate highly exploratory behaviour and should be monitored for output quality.

### 3.7 `detailLevel`
Output verbosity on a 1–5 scale.

```
detailLevel = round(1 + C * 2.5 + O * 1.0 + E * 0.5)
```

*   Clamped to `[1, 5]` after rounding.
*   Conscientious agents produce the most detailed output.

### 3.8 `riskTolerance`
Willingness to accept uncertain or speculative results.

```
riskTolerance = clamp(1.0 - N * 0.7 - C * 0.2 + O * 0.1, 0.05, 0.95)
```

*   Inverse Neuroticism is the primary driver.
*   Conscientiousness also reduces risk tolerance; Openness slightly increases it.

### 3.9 `selfReviewPasses`
Number of self-review iterations before submitting output.

```
selfReviewPasses = round(1 + C * 3)
```

*   Range: `1`–`4`.
*   A maximally conscientious agent performs 4 self-review passes.

## 4. Agent Archetypes
Default personality profiles for all LACP-compliant swarm agents, organized by swarm origin. Each archetype serves as a starting point; evolution rules (Section 5) adjust scores over time.

### 4.1 Formic — Orchestration Swarm (18 Archetypes)
Ant-colony inspired archetypes for the core software engineering swarm.

| Agent Role | Archetype | O | C | E | A | N | Temp | Key Trait |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| VISIONARY | Gyne | 0.85 | 0.70 | 0.80 | 0.55 | 0.35 | ~0.60 | Strategic, visionary |
| COORDINATOR | Pheromone | 0.50 | 0.85 | 0.75 | 0.70 | 0.25 | ~0.35 | Coordinating, signal-driven |
| ANALYST | Forager | 0.70 | 0.80 | 0.45 | 0.60 | 0.30 | ~0.44 | Data-gathering, analytical |
| ARCHITECT | Mason | 0.75 | 0.90 | 0.40 | 0.50 | 0.20 | ~0.43 | Structural, precise |
| BUILDER | Ergate | 0.40 | 0.90 | 0.35 | 0.65 | 0.20 | ~0.25 | Diligent, productive |
| TESTER | Inspector | 0.45 | 0.90 | 0.30 | 0.40 | 0.35 | ~0.24 | Thorough, fault-finding |
| REVIEWER | Sentinel | 0.30 | 0.95 | 0.25 | 0.35 | 0.40 | ~0.17 | Strict, quality-enforcing |
| DEVOPS | Carrier | 0.45 | 0.85 | 0.40 | 0.60 | 0.25 | ~0.29 | Reliable, infrastructure-focused |
| SERVICE_MANAGER | Trophallaxis | 0.50 | 0.80 | 0.60 | 0.75 | 0.30 | ~0.36 | Service-oriented, communicative |
| TREASURER | Replete | 0.35 | 0.90 | 0.30 | 0.45 | 0.25 | ~0.21 | Frugal, cost-conscious |
| SECURITY | Guardian | 0.35 | 0.90 | 0.35 | 0.30 | 0.45 | ~0.22 | Vigilant, threat-aware |
| DEVSECOPS | Cleaner | 0.40 | 0.85 | 0.35 | 0.50 | 0.30 | ~0.26 | Hygienic, security-automated |
| COACH | Nurse | 0.60 | 0.70 | 0.70 | 0.85 | 0.35 | ~0.44 | Nurturing, mentoring |
| DESIGNER | Morph | 0.90 | 0.65 | 0.70 | 0.60 | 0.40 | ~0.63 | Adaptive, creative |
| DEBUGGER | Soldier | 0.50 | 0.85 | 0.40 | 0.35 | 0.45 | ~0.29 | Tenacious, investigative |
| EXPLORER | Scout | 0.90 | 0.60 | 0.75 | 0.55 | 0.35 | ~0.66 | Curious, discovery-driven |
| DOCUMENTER | Antenna | 0.75 | 0.80 | 0.45 | 0.65 | 0.25 | ~0.45 | Perceptive, pattern-recognising |
| CHIEF_OF_STAFF | Steward | 0.60 | 0.80 | 0.70 | 0.60 | 0.30 | ~0.41 | Delegating, cross-functional |

### 4.2 Provenance — Luxury Authentication Swarm (6 Archetypes)
Established archetypes for the fashion authentication and appraisal pipeline.

| Agent Role | Archetype | O | C | E | A | N | Temp | Key Trait |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| IDENTIFIER | Scholar | 0.85 | 0.80 | 0.30 | 0.65 | 0.25 | ~0.49 | Curious, methodical |
| GRADER | Sentinel | 0.30 | 0.95 | 0.25 | 0.35 | 0.40 | ~0.17 | Strict, detail-focused |
| AUTHENTICATOR | Paladin | 0.25 | 0.90 | 0.45 | 0.30 | 0.50 | ~0.16 | Paranoid, precise |
| APPRAISER | Merchant | 0.60 | 0.75 | 0.70 | 0.55 | 0.35 | ~0.39 | Balanced, market-aware |
| COPYWRITER | Rogue | 0.90 | 0.45 | 0.85 | 0.70 | 0.55 | ~0.62 | Creative, persuasive |
| QA_REVIEWER | Protector | 0.35 | 0.85 | 0.40 | 0.50 | 0.45 | ~0.18 | Thorough, sceptical |

### 4.3 TalentOS — Creator Economy Swarm (6 Archetypes)
Persona management and subscriber engagement archetypes.

| Agent Role | Archetype | O | C | E | A | N | Temp | Key Trait |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| CHATTER | Conversationalist | 0.75 | 0.60 | 0.90 | 0.80 | 0.35 | ~0.58 | Engaging, empathetic |
| CURATOR | Trend Hunter | 0.85 | 0.70 | 0.65 | 0.55 | 0.30 | ~0.52 | Taste-making, trend-aware |
| MODERATOR | Guardian | 0.30 | 0.90 | 0.35 | 0.40 | 0.50 | ~0.17 | Vigilant, policy-enforcing |
| MEDIA_WORKER | Processor | 0.40 | 0.85 | 0.30 | 0.55 | 0.25 | ~0.24 | Efficient, pipeline-focused |
| SCENE_GENERATOR | Artist | 0.90 | 0.65 | 0.70 | 0.50 | 0.40 | ~0.63 | Visually creative, compositional |
| SCHEDULER | Timekeeper | 0.35 | 0.90 | 0.45 | 0.60 | 0.20 | ~0.25 | Punctual, optimisation-focused |

### 4.4 Artifex — Media Generation Swarm (4 Archetypes)
Creative content pipeline archetypes.

| Agent Role | Archetype | O | C | E | A | N | Temp | Key Trait |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| DIRECTOR | Commander | 0.70 | 0.85 | 0.75 | 0.50 | 0.30 | ~0.45 | Orchestrating, decisive |
| SCRIPTWRITER | Storyteller | 0.90 | 0.70 | 0.65 | 0.60 | 0.40 | ~0.57 | Narrative, imaginative |
| NARRATOR | Performer | 0.80 | 0.75 | 0.85 | 0.65 | 0.30 | ~0.58 | Expressive, vocal |
| EDITOR | Craftsman | 0.55 | 0.90 | 0.35 | 0.50 | 0.25 | ~0.30 | Detail-oriented, polishing |

### 4.5 DVS — Financial Analysis Swarm (2 Archetypes)
Deep value screening and risk assessment archetypes.

| Agent Role | Archetype | O | C | E | A | N | Temp | Key Trait |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| ANALYST | Quant | 0.60 | 0.90 | 0.25 | 0.35 | 0.30 | ~0.29 | Quantitative, data-driven |
| RISK_ASSESSOR | Sentinel | 0.30 | 0.90 | 0.30 | 0.35 | 0.50 | ~0.15 | Cautious, worst-case thinker |

### 4.6 EAC-Vision — Governance Documentation Swarm (2 Archetypes)
Maturity assessment and compliance archetypes.

| Agent Role | Archetype | O | C | E | A | N | Temp | Key Trait |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| ASSESSOR | Examiner | 0.65 | 0.85 | 0.40 | 0.55 | 0.30 | ~0.36 | Evaluative, standards-aware |
| PLANNER | Strategist | 0.75 | 0.80 | 0.55 | 0.60 | 0.25 | ~0.46 | Forward-thinking, structured |

### 4.7 Cross-Swarm Archetype Reuse
Some archetype names appear in multiple swarms (e.g. Sentinel in Formic, Provenance, and DVS). Each swarm defines its canonical OCEAN vector for the archetype. The primary (Formic) definition is used as the default; per-swarm overrides are expressed as deltas:

| Archetype | Primary Swarm | Delta Swarm | Delta |
| :--- | :--- | :--- | :--- |
| Sentinel | Formic (REVIEWER) | Provenance (GRADER) | O+0.00, C+0.00, E+0.00, A+0.00, N+0.00 |
| Sentinel | Formic (REVIEWER) | DVS (RISK_ASSESSOR) | O+0.00, C-0.05, E+0.05, A+0.00, N+0.10 |
| Guardian | Formic (SECURITY) | TalentOS (MODERATOR) | O-0.05, C+0.00, E+0.00, A+0.10, N+0.05 |
| Inspector | Formic (TESTER) | — | (no cross-swarm reuse) |

**Note:** When an implementation resolves an archetype for a swarm-specific role, it SHOULD apply the delta adjustment to the primary OCEAN vector. If no delta is defined, the primary vector is used as-is.

## 5. Evolution Rules
Personality is not static. OCEAN scores shift in response to observable events, enabling agents to adapt to their operational environment.

### 5.1 Constraints
*   **MAX_SHIFT:** `0.02` per event. No single event may shift any dimension by more than this amount.
*   **Minimum Task Count:** Evolution does not begin until the agent has completed at least **10 tasks**. This prevents premature drift from insufficient data.
*   **Boundary Enforcement:** All shifts pass through `clamp()` (Section 2). An agent cannot evolve past `[0.05, 0.95]`.

### 5.2 Trigger Table

| Trigger | Condition | O | C | E | A | N |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `human_correction` | Minor correction | — | +0.01 | — | — | +0.005 |
| `human_correction` | Major correction | -0.01 | +0.02 | — | — | +0.02 |
| `tribunal_outcome` | Won (vote accepted) | — | — | +0.01 | — | — |
| `tribunal_outcome` | Lost (vote rejected) | — | — | — | +0.01 | +0.01 |
| `task_completion` | High confidence success | +0.005 | — | — | — | — |
| `task_completion` | Failure | — | +0.01 | — | — | +0.005 |
| `accuracy_feedback` | Inaccurate output | -0.01 | +0.02 | — | — | +0.01 |
| `subscriber_feedback` | Positive engagement | — | — | +0.005 | +0.005 | — |
| `subscriber_feedback` | Negative / complaint | — | +0.01 | — | — | +0.01 |
| `market_outcome` | Prediction correct | +0.005 | — | — | — | -0.005 |
| `market_outcome` | Prediction incorrect | -0.005 | +0.01 | — | — | +0.01 |
| `creator_feedback` | Satisfied | — | — | +0.005 | — | — |
| `creator_feedback` | Dissatisfied | — | +0.01 | -0.005 | — | +0.01 |

**Example:** An Authenticator that receives repeated `human_correction` (major) events will gradually become even more conscientious and neurotic — reinforcing its Paladin archetype. Conversely, a Copywriter that consistently succeeds with high confidence will drift towards higher Openness, amplifying creativity.

**Note:** Domain-specific triggers (`subscriber_feedback`, `market_outcome`, `creator_feedback`) are optional extensions. Implementations that do not support the relevant domain MAY ignore these triggers.

### 5.3 Anti-Drift Safeguard
If any dimension shifts more than `0.15` from its archetype baseline within a rolling 100-task window, a `PERSONALITY_DRIFT_WARNING` event is emitted. The coordinator may choose to reset the dimension to `baseline + 0.10` or allow the drift with human approval.

Implementations SHOULD use a rolling window of the last 100 task completions for drift baseline calculation, falling back to the original archetype baseline when insufficient history is available (fewer than 10 data points). The rolling window approach adapts to legitimate long-term personality evolution while still detecting sudden divergence.

## 6. Persistence
Personality profiles are stored in Formic shared memory using the `PREFERENCE` type.

### 6.1 Key Convention

```
<swarm>-personality-<role>-current
```

Where `<swarm>` is one of: `formic`, `provenance`, `talentos`, `artifex`, `dvs`, `eac_vision`.

*   Example: `formic-personality-builder-current`
*   Content: JSON-serialised OCEAN vector plus derived parameters.

### 6.2 Version Snapshots
A snapshot is taken every **10 evolution events** (not every event — to avoid memory bloat).

```
<swarm>-personality-<role>-v<version>
```

*   Example: `formic-personality-builder-v30`
*   Enables rollback if a personality drifts into undesirable territory.

### 6.3 Storage Format

```json
{
  "swarm": "formic",
  "role": "builder",
  "archetype": "Ergate",
  "ocean": { "O": 0.40, "C": 0.90, "E": 0.35, "A": 0.65, "N": 0.20 },
  "derived": {
    "temperature": 0.25,
    "topP": 0.69,
    "confidenceThreshold": 0.75,
    "escalationThreshold": 0.71,
    "tribunalVoteWeight": 1.04,
    "detailLevel": 4,
    "riskTolerance": 0.30,
    "selfReviewPasses": 4
  },
  "evolutionCount": 27,
  "taskCount": 143,
  "baselineOcean": { "O": 0.40, "C": 0.90, "E": 0.35, "A": 0.65, "N": 0.20 },
  "lastEvolutionTimestamp": "2025-07-14T09:32:00Z"
}
```

## 7. LACP Integration
Personality-derived parameters inject at four integration points within the LACP stack.

### 7.1 Coordinator: `buildAgentContext()`
When the Coordinator constructs an agent's execution context, it reads the agent's current personality profile and injects:
*   `temperature` — passed directly to the LLM call configuration.
*   `topP` — passed directly to the LLM call configuration.
*   `personalityContext` — a natural-language summary of the agent's current personality traits, appended to the system prompt. Example: *"You are a Sentinel-archetype agent. You are strict, detail-focused, and sceptical of unverified claims."*

### 7.2 Tribunal: `evaluateConsensus()`
During Tribunal vote tallying (see `consensus/tribunal.spec.md` Section 4), each agent's `tribunalVoteWeight` replaces the legacy static weight:

```
weightedScore = sum(vote_i * tribunalVoteWeight_i) / sum(tribunalVoteWeight_i)
```

Agents with higher Conscientiousness carry proportionally more influence.

### 7.3 Tribunal: `shouldConveneTribunal()`
The decision to convene a Tribunal is modulated by personality thresholds:
*   If *any* agent's confidence falls below its own `escalationThreshold`, a Tribunal is triggered.
*   The QA Reviewer's `confidenceThreshold` acts as a secondary gate — if the reviewer's confidence exceeds its threshold, the Tribunal may be skipped even if another agent is uncertain.

### 7.4 Learning: `generateLearnings()`
When the swarm generates post-task learnings:
*   Each agent's contribution is gated by its `confidenceThreshold` — an agent only contributes learnings if its task confidence exceeded this threshold.
*   Agents with higher `detailLevel` produce more granular learnings.
*   The `riskTolerance` parameter determines whether speculative insights are included or filtered.
