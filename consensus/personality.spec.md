# LACP Consensus: Adaptive Agent Personality System (AAPS)

## 1. Overview
Personality profiles replace hardcoded agent configurations with parametric behaviour derived from the **OCEAN** (Big Five) personality dimensions. Rather than defining each agent's behaviour through static config values, AAPS expresses agent tendencies as a five-dimensional personality vector. Concrete parameters (temperature, confidence thresholds, vote weight) are then *derived* from this vector, enabling emergent behavioural diversity and gradual evolution over time.

**Design Goals:**
*   Eliminate brittle per-agent config files in favour of a single parametric model.
*   Allow agent behaviour to evolve in response to feedback without manual tuning.
*   Maintain deterministic derivation — the same OCEAN scores always produce the same parameters.
*   Preserve backwards compatibility — default OCEAN scores produce behaviour equivalent to legacy static configs.

## 2. OCEAN Dimensions
Each agent carries a five-dimensional personality vector. All dimensions use a continuous `0.0`–`1.0` scale, **bounded to a narrow sub-range** to prevent degenerate behaviour at the extremes.

| Dimension | Code | Low (→ 0.0) | High (→ 1.0) |
| :--- | :--- | :--- | :--- |
| **Openness** | `O` | Conservative, routine-following | Exploratory, creative |
| **Conscientiousness** | `C` | Flexible, fast, loose checks | Meticulous, thorough, slow |
| **Extraversion** | `E` | Reserved, minimal output | Verbose, proactive communication |
| **Agreeableness** | `A` | Contrarian, independent | Deferential, consensus-seeking |
| **Neuroticism** | `N` | Calm, risk-tolerant | Anxious, cautious, escalation-prone |

**Boundary Enforcement:** Any mutation (evolution event, manual override) is clamped to the valid range before persistence.

## 3. Derived Behaviour Parameters
OCEAN scores are mapped to concrete runtime parameters via deterministic formulae. All derivations are pure functions — no hidden state. Implementations define their own weighted linear combinations of OCEAN dimensions for each parameter.

### 3.1 `temperature`
How creative or conservative the agent's LLM sampling is. Primarily driven by Openness (positive) and Conscientiousness (negative).

### 3.2 `topP`
Nucleus sampling diversity. Derived primarily from Openness with secondary influences.

### 3.3 `confidenceThreshold`
Minimum confidence to accept own output without escalation. Driven by Conscientiousness and Neuroticism.

### 3.4 `escalationThreshold`
Confidence floor below which the agent triggers a Tribunal. Inverse Neuroticism is the primary driver.

### 3.5 `tribunalVoteWeight`
Influence weight when casting votes in Tribunal consensus.
*   Conscientiousness is the primary positive driver — meticulous agents carry more weight.
*   Agreeableness acts as a negative modifier — highly agreeable agents carry slightly less weight, reducing their influence on consensus outcomes.

### 3.6 Temperature Calibration Bands
The `temperature` derivation produces values that cluster into four operational bands. Implementations SHOULD use these bands to validate archetype configurations and flag anomalies.

| Band | Range | Use Case |
| :--- | :--- | :--- |
| Conservative | 0.15–0.25 | Precision tasks, auditing, safety |
| Moderate | 0.25–0.45 | Planning, coordination, analysis |
| Balanced | 0.45–0.55 | Documentation, coaching, research |
| Creative | 0.55–0.70 | Design, exploration, strategy |

**Note:** Temperatures above 0.70 are permitted but indicate highly exploratory behaviour and should be monitored for output quality.

### 3.7 `detailLevel`
Output verbosity on a 1–5 scale. Primarily driven by Conscientiousness.

### 3.8 `riskTolerance`
Willingness to accept uncertain or speculative results. Inverse Neuroticism is the primary driver.

### 3.9 `selfReviewPasses`
Number of self-review iterations before submitting output. Range: `1`–`4`. Driven by Conscientiousness.

## 4. Agent Archetypes
Default personality profiles for LACP-compliant swarm agents. Each archetype serves as a starting point; evolution rules (Section 5) adjust scores over time.

Implementations define their own archetype sets appropriate to their domain. LACP defines the following archetype *categories* based on temperature band:

| Category | Representative Names | Typical O | Typical C | Typical N | Band |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Precision** | Sentinel, Inspector, Ergate | Low | Very High | Moderate-High | Conservative |
| **Coordination** | Pheromone, Carrier, Steward | Moderate | High | Low-Moderate | Moderate |
| **Analytical** | Scholar, Antenna, Nurse | Moderate-High | High | Low-Moderate | Balanced |
| **Creative** | Morph, Scout, Gyne, Rogue | High | Moderate | Low | Creative |

### 4.1 Cross-Swarm Archetype Reuse
Some archetype names may appear in multiple swarms. Each swarm defines its canonical OCEAN vector for the archetype. The primary swarm definition is used as the default; per-swarm overrides are expressed as deltas applied to the primary OCEAN vector.

## 5. Evolution Rules
Personality is not static. OCEAN scores shift in response to observable events, enabling agents to adapt to their operational environment.

### 5.1 Constraints
*   **MAX_SHIFT:** Small increment per event (implementation-defined). No single event may shift any dimension beyond this bound.
*   **Minimum Task Count:** Evolution does not begin until the agent has completed a minimum number of tasks. This prevents premature drift from insufficient data.
*   **Boundary Enforcement:** All shifts are clamped to the valid range (Section 2). An agent cannot evolve past the defined bounds.

### 5.2 Trigger Categories

| Trigger | Description | Typical OCEAN Effect |
| :--- | :--- | :--- |
| `human_correction` | Human overrides agent output | Increases C, may increase N |
| `tribunal_outcome` | Agent wins or loses a Tribunal vote | Winner: +E; Loser: +A, +N |
| `task_completion` | Agent completes a task | Success: +O; Failure: +C |
| `accuracy_feedback` | Post-hoc validation of agent output | If inaccurate: -O, +C, +N |

**Note:** Domain-specific triggers (subscriber feedback, market outcomes, creator feedback) are optional extensions. Implementations that support the relevant domain define their own trigger-to-OCEAN mappings.

### 5.3 Anti-Drift Safeguard
If any dimension shifts significantly from its archetype baseline within a rolling task window, a `PERSONALITY_DRIFT_WARNING` event is emitted. The coordinator may choose to reset the dimension or allow the drift with human approval.

## 6. Persistence
Personality profiles are stored in shared memory.

### 6.1 Key Convention

```
<swarm>-personality-<role>-current
```

*   Content: JSON-serialised OCEAN vector plus derived parameters.

### 6.2 Version Snapshots
A snapshot is taken periodically to enable rollback if a personality drifts into undesirable territory.

```
<swarm>-personality-<role>-v<version>
```

### 6.3 Storage Format

```json
{
  "swarm": "example",
  "role": "builder",
  "archetype": "Worker",
  "ocean": { "O": 0.0, "C": 0.0, "E": 0.0, "A": 0.0, "N": 0.0 },
  "derived": {
    "temperature": 0.0,
    "topP": 0.0,
    "confidenceThreshold": 0.0,
    "escalationThreshold": 0.0,
    "tribunalVoteWeight": 0.0,
    "detailLevel": 0,
    "riskTolerance": 0.0,
    "selfReviewPasses": 0
  },
  "evolutionCount": 0,
  "taskCount": 0,
  "baselineOcean": { "O": 0.0, "C": 0.0, "E": 0.0, "A": 0.0, "N": 0.0 },
  "lastEvolutionTimestamp": "2025-01-01T00:00:00Z"
}
```

## 7. LACP Integration
Personality-derived parameters inject at four integration points within the LACP stack.

### 7.1 Coordinator: `buildAgentContext()`
When the Coordinator constructs an agent's execution context, it reads the agent's current personality profile and injects:
*   `temperature` — passed directly to the LLM call configuration.
*   `topP` — passed directly to the LLM call configuration.
*   `personalityContext` — a natural-language summary of the agent's current personality traits, appended to the system prompt.

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
*   Each agent's contribution is gated by its `confidenceThreshold`.
*   Agents with higher `detailLevel` produce more granular learnings.
*   The `riskTolerance` parameter determines whether speculative insights are included or filtered.
