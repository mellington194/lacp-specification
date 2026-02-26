# LACP Layer 3: Tribunal Consensus (Detailed)

## 1. The Tribunal Logic
The Tribunal is the **Governance Engine** for the swarm. It prevents "Single Point of Failure" in decision making.

### 2. The Commit-Reveal Flow (Step-by-Step)

#### Phase 1: Proposal
*   **Agent A** proposes: "Post this tweet."
*   **Router** selects Jury: `[REVIEWER, SECURITY, SAFETY_OS]`

#### Phase 2: Commit (Blind Vote)
*   **Reviewer** thinks "Approves". Generates Salt `xyz123`.
    *   Sends Hash: `SHA256("APPROVE" + "xyz123")` -> `a1b2c3...`
*   **Security** thinks "Reject". Generates Salt `abc987`.
    *   Sends Hash: `SHA256("REJECT" + "abc987")` -> `d4e5f6...`
*   *State:* The Router has hashes, but doesn't know the votes. No agent knows what the other voted.

#### Phase 3: Reveal
*   **Router** signals: "All votes in. Open keys."
*   **Reviewer** sends: `{"vote": "APPROVE", "salt": "xyz123"}`.
*   **Security** sends: `{"vote": "REJECT", "salt": "abc987"}`.

#### Phase 4: Tally & Sycophancy Check
*   **Router** runs sycophancy detection (see `consensus/sycophancy.spec.md`):
    1.  Pairwise similarity: discard derivative votes (similarity >= configurable threshold, default 0.92).
    2.  Cluster detection: identify and resolve sycophantic clusters (3+ agents with mean similarity above warning threshold).
*   **If** `Similarity(Reviewer, Security) >= derivativeThreshold`:
    *   Discard the vote of the agent with the lower `tribunalVoteWeight`.
*   **Final Count:** 1 APPROVE vs 1 REJECT.
    *   **Result:** `REJECT` (Safety Default).

## 3. Mathematical Model (Sycophancy)

$$ \text{SycophancyScore} = \max_{i \neq j} \left( \frac{\mathbf{v}_i \cdot \mathbf{v}_j}{\|\mathbf{v}_i\| \|\mathbf{v}_j\|} \right) $$

Where $\mathbf{v}_i$ is the embedding vector of Agent $i$'s `reasoning` text.

*   **Safe Zone:** Score < `warningThreshold` (default: 0.80).
*   **Warning:** `warningThreshold` <= Score < `derivativeThreshold` (default: 0.80 - 0.92). Flagged for audit.
*   **Derivative:** Score >= `derivativeThreshold` (default: 0.92). Vote discarded.
*   **Cluster:** 3+ agents in warning zone with high mean density. Follower votes discarded.

See `consensus/sycophancy.spec.md` for full algorithm, cluster detection, and tuning guidelines.

## 4. Dynamic Vote Weighting
Legacy Tribunal tallying used static, role-based vote weights (e.g., Security = 1.2, Reviewer = 1.0). AAPS replaces this with **personality-derived weights** that evolve alongside the agent.

### 4.1 Weight Derivation
Each agent's `tribunalVoteWeight` is derived from the **Conscientiousness** dimension of its OCEAN personality vector using a linear mapping. Higher Conscientiousness produces higher vote weight, reflecting the principle that meticulous agents should have proportionally greater influence in consensus decisions.

### 4.2 Weighted Tally
The weighted tally replaces simple vote counting. Each vote is scaled by the casting agent's weight:

```
weightedScore = sum(vote_i * weight_i) / sum(weight_i)
```

Where `vote_i` is `+1` (APPROVE) or `-1` (REJECT), and `weight_i` is the agent's `tribunalVoteWeight`.

*   **Threshold:** `weightedScore > 0.0` → APPROVE; `weightedScore <= 0.0` → REJECT (Safety Default).
*   A tie or exact zero defaults to REJECT, preserving the existing safety-first principle.

### 4.3 Backwards Compatibility
Default OCEAN scores produce weights in the range `~0.85`–`~1.15` for all standard archetypes. When all agents have similar Conscientiousness, the weighted tally is functionally equivalent to a simple majority vote — ensuring no behavioural change for swarms that have not yet adopted AAPS.

### 4.4 Weight Evolution
Weights are not static. As an agent's personality evolves (see `consensus/personality.spec.md` Section 5), its vote weight shifts accordingly:
*   An agent that repeatedly **loses** Tribunal votes gains Agreeableness and Neuroticism, which do not directly affect weight — but repeated `accuracy_feedback` events increase Conscientiousness, which *does*.
*   An agent that is consistently overruled will gradually become more deferential (higher Agreeableness), reducing its likelihood of dissenting in the first place.

## 5. Extensible Trigger Conditions
Tribunal convocation is no longer limited to a generic confidence threshold. Domain-specific triggers can be registered per-swarm via the `TribunalTriggerRegistry`.

### 5.1 Trigger Registry
Each swarm registers a set of named triggers. A trigger is a predicate function that receives the current task context and returns a boolean.

```typescript
interface TribunalTrigger {
  name: string;
  description: string;
  evaluate(context: TaskContext): boolean;
  priority: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
}
```

Triggers are evaluated after each agent completes its task. If *any* trigger returns `true`, a Tribunal is convened.

### 5.2 Example Triggers

| # | Trigger | Condition | Priority |
| :--- | :--- | :--- | :--- |
| 1 | `low_confidence` | Any agent's confidence below threshold | MEDIUM |
| 2 | `inter_agent_conflict` | Two agents produce contradictory assessments | HIGH |
| 3 | `high_stakes_decision` | Value or risk exceeds configured threshold | HIGH |
| 4 | `unknown_input` | Low match score against known pattern database | MEDIUM |
| 5 | `ambiguous_classification` | Agent uncertain about category or severity | MEDIUM |

### 5.3 Personality Modulation of Triggers
Trigger thresholds are not fixed — they are modulated by the personalities of the agents involved:
*   **Trigger 1 (Low Confidence):** The 0.6 threshold is a *base*. The QA Reviewer's `escalationThreshold` (derived from Neuroticism) may lower this further for cautious agents, or raise it for risk-tolerant ones.
*   **Trigger 2 (Grade/Auth Conflict):** The Grader's `confidenceThreshold` determines how certain the grader must be to override the Authenticator's concern. A highly conscientious Grader with a high confidence threshold is less likely to trigger this — it would only fire on genuine disagreement, not marginal cases.
*   **Triggers 3–5:** These are domain-specific and use fixed thresholds, but the *decision to act* on the Tribunal outcome is personality-influenced (Section 6).

### 5.4 Custom Registration
Swarms register triggers at initialisation:

```typescript
TribunalTriggerRegistry.register('my-swarm', [
  new LowConfidenceTrigger(0.6),
  new InterAgentConflictTrigger(),
  new HighStakesTrigger(5000),
  new UnknownInputTrigger(0.3),
  new AmbiguousClassificationTrigger(),
]);
```

Triggers can be added, removed, or modified at runtime without restarting the swarm.

## 6. Personality-Modulated Consensus
The interplay between OCEAN personality dimensions and Tribunal behaviour creates emergent consensus dynamics that go beyond simple voting.

### 6.1 Neuroticism → Escalation Frequency
An agent with high Neuroticism has a lower `escalationThreshold`. This means:
*   It triggers Tribunals more often, even for moderate uncertainty.
*   The swarm benefits from this "canary" behaviour — the most anxious agent surfaces edge cases that calmer agents might overlook.
*   Over time, if escalations prove unnecessary (agent wins Tribunal repeatedly), Neuroticism drifts down slightly, reducing false positives.

### 6.2 Conscientiousness → Vote Influence
An agent with high Conscientiousness carries more weight in Tribunal votes. Its `tribunalVoteWeight` means its vote counts significantly more than a low-Conscientiousness agent.
*   This is by design — the most meticulous agents should have the strongest voice in quality decisions.

### 6.3 Agreeableness → Deference
An agent with high Agreeableness tends to defer to consensus:
*   If the preliminary tally (before the agent's own vote) already shows a strong majority, a highly agreeable agent may **abstain** rather than dissent.
*   Abstention threshold: `abs(preliminaryScore) > agreeableness * 0.8`.
*   This reduces noise in Tribunal outcomes but preserves strong independent objections (an agent will still vote if it has high confidence and low Agreeableness).

### 6.4 Combined Effects
The personality dimensions interact to produce nuanced behaviour:
*   **High N + High C** (e.g., QA Reviewer): Escalates often *and* carries heavy weight — the "quality gatekeeper."
*   **Low N + High O** (e.g., Copywriter): Rarely escalates, tolerates uncertainty, but when forced into a Tribunal, votes creatively.
*   **High N + Low C** (unlikely in standard archetypes): Would escalate frequently but carry little weight — a "worrier without authority."
*   **High A + High C** (e.g., Identifier): Defers to consensus but, when the consensus is weak, steps in with authoritative weight.

### 6.5 Cross-Reference
See `consensus/personality.spec.md` for the full OCEAN derivation rules, archetype definitions, and evolution mechanics.