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
tribunalVoteWeight = 0.5 + C * 0.7
```

*   Range: `0.5`–`1.15`.
*   Conscientiousness is the sole driver — meticulous agents carry more weight.

### 3.6 `detailLevel`
Output verbosity on a 1–5 scale.

```
detailLevel = round(1 + C * 2.5 + O * 1.0 + E * 0.5)
```

*   Clamped to `[1, 5]` after rounding.
*   Conscientious agents produce the most detailed output.

### 3.7 `riskTolerance`
Willingness to accept uncertain or speculative results.

```
riskTolerance = clamp(1.0 - N * 0.7 - C * 0.2 + O * 0.1, 0.05, 0.95)
```

*   Inverse Neuroticism is the primary driver.
*   Conscientiousness also reduces risk tolerance; Openness slightly increases it.

### 3.8 `selfReviewPasses`
Number of self-review iterations before submitting output.

```
selfReviewPasses = round(1 + C * 3)
```

*   Range: `1`–`4`.
*   A maximally conscientious agent performs 4 self-review passes.

## 4. Agent Archetypes
Default personality profiles for Provenance swarm agents. These serve as starting points; evolution rules (Section 5) adjust scores over time.

| Agent | Archetype | O | C | E | A | N | Temperature | Key Trait |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Identifier | Scholar | 0.80 | 0.75 | 0.50 | 0.60 | 0.30 | ~0.28 | Curious, methodical |
| Grader | Sentinel | 0.40 | 0.90 | 0.35 | 0.30 | 0.55 | ~0.14 | Strict, detail-focused |
| Authenticator | Paladin | 0.20 | 0.85 | 0.30 | 0.20 | 0.80 | ~0.05 | Paranoid, precise |
| Appraiser | Merchant | 0.55 | 0.70 | 0.65 | 0.50 | 0.35 | ~0.32 | Balanced, market-aware |
| Copywriter | Rogue | 0.90 | 0.45 | 0.85 | 0.70 | 0.25 | ~0.82 | Creative, persuasive |
| QA Reviewer | Protector | 0.30 | 0.95 | 0.40 | 0.25 | 0.70 | ~0.06 | Thorough, sceptical |

**Reading the Table:**
*   The Authenticator (Paladin) has the lowest temperature (~0.05) — nearly deterministic output, maximum paranoia.
*   The Copywriter (Rogue) has the highest temperature (~0.82) — highly creative, willing to take risks.
*   The Grader (Sentinel) and QA Reviewer (Protector) both have high Conscientiousness but differ in Neuroticism — the Protector is more anxious and escalates more aggressively.

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

**Example:** An Authenticator that receives repeated `human_correction` (major) events will gradually become even more conscientious and neurotic — reinforcing its Paladin archetype. Conversely, a Copywriter that consistently succeeds with high confidence will drift towards higher Openness, amplifying creativity.

### 5.3 Anti-Drift Safeguard
If any dimension shifts more than `0.15` from its archetype baseline within a rolling 100-task window, a `PERSONALITY_DRIFT_WARNING` event is emitted. The coordinator may choose to reset the dimension to `baseline + 0.10` or allow the drift with human approval.

## 6. Persistence
Personality profiles are stored in Formic shared memory using the `PREFERENCE` type.

### 6.1 Key Convention

```
provenance-personality-<role>-current
```

*   Example: `provenance-personality-grader-current`
*   Content: JSON-serialised OCEAN vector plus derived parameters.

### 6.2 Version Snapshots
A snapshot is taken every **10 evolution events** (not every event — to avoid memory bloat).

```
provenance-personality-<role>-v<version>
```

*   Example: `provenance-personality-grader-v30`
*   Enables rollback if a personality drifts into undesirable territory.

### 6.3 Storage Format

```json
{
  "role": "grader",
  "archetype": "Sentinel",
  "ocean": { "O": 0.40, "C": 0.90, "E": 0.35, "A": 0.30, "N": 0.55 },
  "derived": {
    "temperature": 0.14,
    "topP": 0.67,
    "confidenceThreshold": 0.74,
    "escalationThreshold": 0.56,
    "tribunalVoteWeight": 1.13,
    "detailLevel": 4,
    "riskTolerance": 0.27,
    "selfReviewPasses": 4
  },
  "evolutionCount": 27,
  "taskCount": 143,
  "baselineOcean": { "O": 0.40, "C": 0.90, "E": 0.35, "A": 0.30, "N": 0.55 },
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
