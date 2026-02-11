# Method and System for Secure Multi-Agent Communication and Consensus (LACP)

## Defensive Publication

> **Establishing Prior Art for Agent-to-Agent Communication Standards**

---

**Publication Type**: Defensive Disclosure
**Publication Date**: January 29, 2026 (Original)
**Revision**: 3.0 (February 2026)
**Author**: Envalr Engineering Team
**Archive**: Zenodo ([10.5281/zenodo.18614115](https://doi.org/10.5281/zenodo.18614115)), arXiv (submission pending)
**Status**: Draft Standard v1.0
**Field:** Artificial Intelligence / Distributed Systems
**Keywords:** Multi-Agent Systems, AI Governance, Consensus Algorithms, Sycophancy Detection, Prompt Injection Security, OCEAN Personality Model, Adaptive Agent Personality, Cross-Swarm Learning

---

## Abstract

This document constitutes a defensive publication establishing prior art for the **Lightweight Agentic Communication Protocol (LACP)**, a Layer 7 application protocol designed for secure, typed communication between autonomous AI agents in multi-agent systems (MAS). LACP addresses critical vulnerabilities in current agent frameworks including prompt injection propagation, sycophancy cascades, mission drift, and lack of consensus mechanisms for high-stakes decisions.

The protocol implements a three-layer architecture: **Transport** (rate limiting, injection scanning, PII redaction, encryption), **Semantic** (context objects, immutable objectives, drift detection, constraint propagation), and **Consensus** (blind voting via commit-reveal, sycophancy detection, personality-weighted consensus). Version 2 extends the specification with: (a) the **Adaptive Agent Personality System (AAPS)**, which uses OCEAN psychological dimensions to derive agent behavioural parameters; (b) **personality-weighted voting** in the Tribunal consensus mechanism; (c) **cross-swarm learning** via the Gene Pool service; and (d) domain-specific agent archetypes for specialised swarms.

By publishing this specification, Envalr establishes prior art to prevent third parties from obtaining patents on these fundamental architectural patterns, while retaining the right to implement and commercialize LACP-compliant systems.

---

## 1. Problem Statement

### 1.1 Current State of Agent Communication

As Large Language Models (LLMs) are deployed in autonomous agent roles, existing multi-agent frameworks (LangChain, AutoGen, CrewAI, Microsoft Semantic Kernel) lack standardized protocols for agent-to-agent communication. Agents typically communicate via:

- Unstructured natural language strings
- Ad-hoc JSON message formats
- Direct function calls without validation

### 1.2 Identified Vulnerabilities

| Vulnerability | Description | Impact |
|--------------|-------------|--------|
| **Prompt Injection Propagation** | Malicious prompts traverse agent chains undetected | System compromise, data exfiltration |
| **Sycophancy Cascades** | Agents reinforce each other's errors through agreement bias | Incorrect outputs, hallucination amplification |
| **Mission Drift** | Deviation from original objective over long agent chains | Task failure, resource waste |
| **Lack of Consensus** | No mechanism for multi-agent agreement on high-stakes actions | Unauthorized actions, inconsistent state |
| **No Rate Limiting** | Runaway agents can exhaust resources | DoS, cost explosion |
| **PII Leakage** | Sensitive data passes between agents without filtering | Privacy violations, compliance failures |
| **Static Behaviour** | Agent parameters (temperature, thresholds, voting weights) are hardcoded | Brittle systems requiring code changes for recalibration |

---

## 2. LACP Architecture Overview

LACP implements a three-layer architecture, routing all inter-agent traffic through a centralized **Agent Router** to enforce a "Zero-Trust" environment:

```
+--------------------------------------------------+
|              Layer 3: CONSENSUS                  |
|    (Blind Voting, Sycophancy Detection,          |
|     Personality-Weighted Consensus)              |
+--------------------------------------------------+
|              Layer 2: SEMANTIC                   |
|    (Context Objects, Drift Prevention,           |
|     Constraint Propagation)                      |
+--------------------------------------------------+
|              Layer 1: TRANSPORT                  |
|    (Agent Router, Security, Rate Limiting,       |
|     Injection Scanning, PII Redaction)           |
+--------------------------------------------------+
|              Underlying Transport                |
|         (gRPC, HTTP/2, WebSocket)                |
+--------------------------------------------------+
```

---

## 3. Layer 1: Transport & Security Layer

### 3.1 Agent Router (Mandatory Gateway)

All inter-agent communication MUST traverse a centralized Agent Router. Direct peer-to-peer connections between agents are prohibited.

**Router Responsibilities**:
- Message routing and delivery confirmation
- Authentication and authorization
- Rate limiting enforcement
- Security scanning
- Audit logging

### 3.2 Message Envelope Format

```json
{
  "lacp_version": "1.0",
  "message_id": "uuid-v4",
  "timestamp": "ISO-8601",
  "source_agent_id": "string",
  "destination_agent_id": "string | broadcast",
  "message_type": "REQUEST | RESPONSE | EVENT | VOTE",
  "priority": "LOW | NORMAL | HIGH | CRITICAL",
  "ttl_ms": 30000,
  "correlation_id": "uuid-v4",
  "headers": { },
  "payload": { }
}
```

### 3.3 Security Enforcements

#### 3.3.1 Rate Limiting

Token-bucket algorithm per `agent_id`:
- Default: 100 requests/minute
- Configurable per agent role
- Burst allowance: 2x base rate for 10 seconds

#### 3.3.2 Injection Scanning

Every text payload is scored against a repository of known injection patterns:
- Pattern matching against injection database
- LLM-based semantic analysis for novel patterns
- Threshold: Payloads with >0.8 injection probability are dropped
- Dropped messages trigger security alert

#### 3.3.3 PII Redaction

Outbound payloads pass through Named Entity Recognition (NER):

| Entity Type | Replacement Token |
|-------------|-------------------|
| Credit Card | `<REDACTED_PII_CC>` |
| SSN | `<REDACTED_PII_SSN>` |
| Email | `<REDACTED_PII_EMAIL>` |
| Phone | `<REDACTED_PII_PHONE>` |
| Address | `<REDACTED_PII_ADDR>` |
| Name | `<REDACTED_PII_NAME>` |

#### 3.3.4 Encryption

- All messages encrypted in transit (TLS 1.3+)
- Optional end-to-end encryption between specific agents
- Key rotation every 24 hours

---

## 4. Layer 2: Semantic Context Layer

### 4.1 Context Object (Mandatory Header)

Every LACP packet MUST include a typed `Context` object:

```json
{
  "protocol": "LACP/1.0",
  "mission_id": "uuid-v4",
  "trace_id": "uuid-v4",
  "sequence_number": 42,
  "objective": {
    "original": "Immutable original user prompt",
    "hash": "sha256-of-original"
  },
  "constraints": [
    "Do not delete files",
    "Do not access external internet",
    "Budget limit: $0.50"
  ],
  "history_vector": "base64_encoded_embedding",
  "parent_context_hash": "sha256-of-parent",
  "payload": { }
}
```

### 4.2 Immutable Objective

The `objective.original` field is cryptographically sealed:
- Hash computed at mission start
- Every message includes hash for verification
- Agents CANNOT modify the objective
- Objective tampering triggers mission abort

### 4.3 History Vector

Compressed semantic representation of mission trajectory:
- Embedding of key decisions and state changes
- Enables new agents to "onboard" instantly
- Prevents context window exhaustion
- Updated at each significant state change

### 4.4 Constraint Propagation

Constraints flow downstream and can only become MORE restrictive:
- Child agents inherit parent constraints
- New constraints can be added, never removed
- Budget constraints decrement, never increment

### 4.5 Drift Detection

The Router monitors for objective drift:

```
drift_score = 1 - cosine_similarity(
    current_action_embedding,
    objective_embedding
)

if drift_score > 0.3:
    trigger_drift_alert()
if drift_score > 0.5:
    pause_mission()
```

---

## 5. Layer 3: Consensus Layer (Tribunal)

### 5.1 When Consensus is Required

High-stakes actions requiring multi-agent agreement:
- Destructive operations (delete, overwrite)
- External communications (email, API calls)
- Financial transactions
- User-facing content publication
- Security-sensitive operations

### 5.2 Blind Voting Protocol (Commit-Reveal)

To prevent sycophancy (agents copying each other's votes):

```
Phase 1 - COMMIT (5 second window)
  Agent submits: hash(vote + salt)

Phase 2 - LOCK
  Router confirms receipt of all required votes
  No new commits accepted

Phase 3 - REVEAL (5 second window)
  Agent submits: vote + salt
  Router validates: hash(vote + salt) == committed_hash

Phase 4 - TALLY
  Router counts votes
  Result broadcast to all participants
```

### 5.3 Sycophancy Detection

Post-reveal analysis of voting rationale:

```
for each pair (Agent_A, Agent_B):
    similarity = cosine_similarity(
        Agent_A.reasoning_vector,
        Agent_B.reasoning_vector
    )

    if similarity > 0.95:
        # Votes are suspiciously similar
        # Secondary check: Jaccard similarity on tokenised reasoning
        jaccard = jaccard_similarity(
            tokenize(Agent_A.reasoning_text),
            tokenize(Agent_B.reasoning_text)
        )
        if jaccard > 0.85:
            # Confirmed derivative — discard lower-weight vote
            discard_vote(agent_with_lower_tribunal_weight)
            flag_as_derivative()
```

The agent whose vote is discarded is selected by `tribunalVoteWeight` (see §7.2). In the absence of personality data, the agent with the lower historical confidence score (based on Pheromone decay) is used as the tiebreaker.

### 5.4 Quorum Requirements

| Action Category | Minimum Voters | Required Majority |
|----------------|----------------|-------------------|
| Read-only | 1 | N/A |
| State modification | 2 | Simple majority |
| Destructive | 3 | 2/3 supermajority |
| Security-critical | 3 | Unanimous |

### 5.5 Role-Weighted Voting

Certain agent roles have elevated voting power for specific categories:

| Role | Security Weight | Quality Weight | Business Weight |
|------|-----------------|----------------|-----------------|
| Sentinel | 2.0x | 1.0x | 0.5x |
| Guardian | 1.5x | 1.0x | 1.0x |
| Reviewer | 1.0x | 2.0x | 1.0x |
| Executive | 0.5x | 1.0x | 2.0x |

---

## 6. Adaptive Agent Personality System (AAPS)

LACP v2 introduces the **Adaptive Agent Personality System (AAPS)**, which replaces hardcoded agent parameters with a psychometric derivation model based on the **OCEAN (Big Five)** personality framework.

### 6.1 OCEAN Personality Model

Each agent maintains a `PersonalityProfile` comprising five dimensions, each scored on a continuous [0.0, 1.0] scale:

| Dimension | Symbol | Agent Behaviour Influence |
|-----------|--------|--------------------------|
| **Openness** | O | Creativity, exploration, temperature |
| **Conscientiousness** | C | Precision, thoroughness, confidence thresholds |
| **Extraversion** | E | Assertiveness, voting influence, detail verbosity |
| **Agreeableness** | A | Deference, conflict avoidance, vote weight reduction |
| **Neuroticism** | N | Caution, escalation sensitivity, self-review frequency |

Bounds are enforced at [0.05, 0.95] to prevent degenerate behaviour (e.g., temperature=0 or threshold=1.0).

### 6.2 OCEAN → Derived Behaviour Mapping

The `deriveFromOcean(role, ocean)` function computes operational parameters from OCEAN scores using weighted formulas with role-specific base values:

*   **Temperature**: `baseTemp + (O - 0.5) × 0.6 - (C - 0.5) × 0.3 + (N - 0.5) × 0.1`
*   **Top-P**: `0.9 + (O - 0.5) × 0.15 - (N - 0.5) × 0.05`
*   **Confidence Threshold**: `baseThreshold + (C - 0.5) × 0.3 + (N - 0.5) × 0.15 - (A - 0.5) × 0.1`
*   **Tribunal Vote Weight**: `1.0 + (E - 0.5) × 0.4 + (C - 0.5) × 0.3 - (A - 0.5) × 0.2`, clamped to [0.5, 1.5]
*   **Escalation Threshold**: Derived from N and C, governing when tribunals are convened
*   **Detail Level**: Discrete {concise, standard, verbose} from E + C
*   **Risk Tolerance**: Discrete {conservative, balanced, aggressive} from O, N, C
*   **Self-Review Passes**: Integer [0, 2] from N + C

Default OCEAN scores are calibrated so that derived parameters approximate existing hardcoded values, enabling zero-behaviour-change deployment.

### 6.3 Agent Archetypes

Agents are initialised with **archetypes** — named personality templates that encode domain-appropriate OCEAN scores. Example archetypes from the Provenance luxury authentication swarm:

| Agent Role | Archetype | O | C | E | A | N | Derived Temp |
|-----------|-----------|-----|-----|-----|-----|-----|-------------|
| Identifier | Scholar | 0.80 | 0.75 | 0.65 | 0.60 | 0.30 | ~0.28 |
| Grader | Sentinel | 0.40 | 0.90 | 0.50 | 0.30 | 0.55 | ~0.14 |
| Authenticator | Paladin | 0.20 | 0.85 | 0.45 | 0.20 | 0.80 | ~0.05 |
| Appraiser | Merchant | 0.55 | 0.70 | 0.60 | 0.50 | 0.35 | ~0.32 |
| Copywriter | Rogue | 0.90 | 0.45 | 0.85 | 0.55 | 0.25 | ~0.82 |
| QA Reviewer | Protector | 0.25 | 0.95 | 0.40 | 0.25 | 0.70 | ~0.06 |

Archetypes are domain-specific. Different swarms (e.g., code review, content moderation, financial analysis) define their own archetype sets with appropriate OCEAN calibrations.

### 6.4 Personality Evolution

Agent personalities evolve conservatively based on feedback signals:

| Trigger | OCEAN Shift | Max Δ per Event |
|---------|-------------|-----------------|
| Human correction (major) | +C, +N, -O | ±0.02 |
| Tribunal vote won | +E | ±0.02 |
| Tribunal vote lost | +A, +N | ±0.02 |
| Task completion (high confidence) | +O | ±0.02 |
| Task completion (failure) | +C | ±0.02 |
| Accuracy feedback (inaccurate) | +C, -O | ±0.02 |

**Safeguards:**
*   Minimum 10 completed tasks before evolution begins (cold-start protection)
*   Maximum shift of ±0.02 per event (matching existing `THRESHOLD_ADJUSTMENT_STEP`)
*   OCEAN bounds [0.05, 0.95] enforced after each evolution step
*   Version snapshots every 10 evolutions for rollback capability

### 6.5 Personality Persistence

Personality profiles are persisted to shared memory using a key convention:
*   `personality-<role>-current` — Active profile
*   `personality-<role>-baseline` — Original archetype (immutable)
*   `personality-<role>-v<N>` — Version snapshots at every 10th evolution

This enables profiles to survive process restarts and to be shared across swarm instances.

---

## 7. Personality-Weighted Consensus

LACP v2 extends the Tribunal mechanism (§5) with **personality-weighted voting**, where each agent's vote is multiplied by its derived `tribunalVoteWeight` (range [0.5, 1.5]).

### 7.1 Weighted Consensus Algorithm

Given agents $A_1, A_2, \ldots, A_n$ with vote weights $w_1, w_2, \ldots, w_n$ and verdicts $v_i \in \{\text{APPROVE}, \text{REJECT}, \text{ABSTAIN}\}$:

$$\text{WeightedApproval} = \frac{\sum_{i: v_i = \text{APPROVE}} w_i}{\sum_{i: v_i \neq \text{ABSTAIN}} w_i}$$

Consensus is reached when $\text{WeightedApproval} \geq \theta$, where the default threshold $\theta = 0.67$ (two-thirds supermajority).

### 7.2 Weight Derivation

Vote weight is derived from personality OCEAN scores (see §6.2):
*   **Extraversion** increases voting power — assertive agents carry more influence
*   **Conscientiousness** increases voting power — reliable agents carry more weight
*   **Agreeableness** decreases voting power — deferential agents yield to stronger opinions

This ensures that agents with a track record of independent, careful analysis have proportionally greater influence in consensus decisions, while agents prone to agreement bias are appropriately dampened.

### 7.3 Fallback

The system provides both unweighted (`evaluateConsensus`) and weighted (`evaluateWeightedConsensus`) methods. If personality profiles are unavailable, all agents receive a default weight of 1.0, preserving backward compatibility.

### 7.4 Tribunal Trigger Types

Tribunals are convened automatically based on domain-specific conditions, modulated by personality-derived thresholds:

| Trigger Type | Condition | Personality Influence |
|-------------|-----------|----------------------|
| `grade_review` | Agent confidence < 0.75 | QA reviewer's `escalationThreshold` |
| `grade_vs_auth` | Grader/Authenticator conflict | Both agents' `confidenceThreshold` |
| `high_value_deliberation` | Item value ≥ threshold (e.g., £5,000) | Appraiser's `riskTolerance` |
| `rare_item_review` | Unknown or rare subject | Identifier's `dataRequestProbability` |
| `defect_assessment` | Ambiguous defect grading | Grader's `escalationThreshold` |

---

## 8. Cross-Swarm Learning (Gene Pool)

LACP v2 introduces a **Gene Pool** service that enables cross-swarm knowledge transfer at the protocol level.

### 8.1 Architecture

The Gene Pool operates as a shared knowledge repository using PostgreSQL with `pgvector` for semantic similarity search:

*   **Pattern Storage:** Successful strategies, error patterns, and calibration data are stored as vectorised entries with fitness scores.
*   **Fitness Scoring:** Patterns are scored based on: success rate, adoption count, recency, and cross-swarm applicability.
*   **Sync Protocol:** Swarms periodically push local learnings and pull high-fitness patterns from the shared pool via REST API.

### 8.2 LACP Integration Points

The Gene Pool extends the LACP Transport Layer (§3) with:
*   **Learning Emission:** After tribunal resolution, the winning strategy and its context are emitted as a learning event.
*   **Pattern Ingestion:** On swarm initialisation, high-fitness patterns from the Gene Pool are loaded into the swarm's local context.
*   **Cross-Swarm Personality Calibration:** Personality evolution data (§6.4) from one swarm can inform initial archetype tuning in related swarms.

### 8.3 Trust Boundaries

Gene Pool data inherits the LACP trust model:
*   Patterns from the same organisation receive `INTERNAL` trust level
*   Cross-organisation patterns (future) would receive `API` trust level with additional validation

---

## 9. JSON-RPC Integration

LACP is designed to complement JSON-RPC 2.0 for tool invocation.

### 9.1 Tool Call Wrapping

```json
{
  "lacp_version": "1.0",
  "message_type": "REQUEST",
  "payload": {
    "jsonrpc": "2.0",
    "method": "tools/call",
    "params": {
      "name": "bash",
      "arguments": {
        "command": "git status"
      }
    },
    "id": "request-123"
  }
}
```

### 9.2 MCP Compatibility

LACP messages can encapsulate Model Context Protocol (MCP) payloads:
- Tools, Resources, Prompts wrapped in LACP envelopes
- Security scanning applied before MCP processing
- Context propagation through MCP tool chains

---

## 10. Error Handling

### 10.1 Error Categories

| Code Range | Category | Example |
|------------|----------|---------|
| 1xxx | Transport | Connection timeout, rate limited |
| 2xxx | Security | Injection detected, unauthorized |
| 3xxx | Semantic | Invalid context, drift detected |
| 4xxx | Consensus | Quorum not reached, vote rejected |
| 5xxx | System | Internal error, service unavailable |

### 10.2 Error Response Format

```json
{
  "lacp_version": "1.0",
  "message_type": "RESPONSE",
  "error": {
    "code": 2001,
    "category": "SECURITY",
    "message": "Potential injection pattern detected",
    "details": {
      "pattern_id": "INJ-042",
      "confidence": 0.87,
      "blocked_segment": "[REDACTED]"
    },
    "remediation": "Review and sanitize input"
  }
}
```

---

## 11. Domain Application: Luxury Product Authentication Pipeline

This section describes a concrete LACP deployment: the **Provenance** luxury resale authentication swarm, a 6-agent pipeline that processes product images through identification, grading, authentication, appraisal, copywriting, and quality assurance.

### 11.1 Pipeline Architecture

The swarm employs six specialised agents in a directed acyclic graph with selective parallelism:

```
Stage 1: IDENTIFIER (solo)
    ↓
Stage 2: GRADER + AUTHENTICATOR (parallel)
    ↓
Stage 3: APPRAISER (depends on Stages 1+2)
    ↓
Stage 4: COPYWRITER (depends on Stages 1-3)
    ↓
Stage 5: QA_REVIEWER (depends on all above)
```

Each agent maps to a Formic role that determines its LACP trust level and communication permissions:

| Agent | Formic Role | Responsibility | Model Tier |
|-------|-------------|----------------|------------|
| Identifier | EXPLORER | Brand, model, material, colour detection | Flash (temp 0.2) |
| Grader | WORKER | Dual-standard condition assessment (TRR + Vestiaire Collective) | Flash (temp 0.2) |
| Authenticator | VALIDATOR | Brand-specific authenticity verification, OCR extraction | Pro (temp 0.1) |
| Appraiser | WORKER | GBP resale price estimation with market factor analysis | Flash (temp 0.3) |
| Copywriter | WORKER | SEO-optimised listing title, description, and tags | Flash (temp 0.7) |
| QA Reviewer | VALIDATOR | Cross-agent consistency validation, human review gating | Pro (temp 0.1) |

VALIDATOR-role agents (Authenticator, QA Reviewer) use the higher-accuracy Pro model tier, reflecting the LACP principle that verification agents warrant greater computational investment.

### 11.2 Agent Capability Dependencies

Each agent declares its input requirements and output provisions, enabling the SwarmCoordinator to calculate the execution DAG at runtime:

| Agent | Depends On | Provides | Parallel With |
|-------|-----------|----------|---------------|
| Identifier | (none) | brand, model, material, colour, category | — |
| Grader | Identifier | trrGrade, vcGrade, defects | Authenticator |
| Authenticator | Identifier | indicators, recommendation, serialCode | Grader |
| Appraiser | Identifier, Grader, Authenticator | estimatedResalePrice, priceRange | — |
| Copywriter | All above | title, description, tags | — |
| QA Reviewer | All above | approved, issues, corrections | — |

### 11.3 Visual Embedding Integration (FashionSigLIP)

The pipeline extends LACP Context Objects (§4) with visual embeddings generated by FashionSigLIP, a domain-specific vision-language model trained on fashion product images. These embeddings serve two purposes:

1. **Image Clustering**: Pre-pipeline grouping of uploaded images into product lots using graph-based connected components with cosine similarity thresholds. Sequential filename detection provides additional similarity boosts (+0.30) for consecutively numbered images from the same folder.

2. **Agent Context Enrichment**: Embedding vectors are included in the `history_vector` field of the LACP Context Object, providing a compressed visual signature that allows downstream agents to reason about product identity without re-processing images.

### 11.4 Domain-Specific Tribunal Triggers

The Tribunal mechanism (§5) is extended with five domain-specific trigger types, each convening a panel of three specialist perspectives:

| Trigger | Condition | Panel Composition | Escalates to Human |
|---------|-----------|-------------------|-------------------|
| `grade_review` | Agent confidence < 0.75 | condition_specialist, defect_analyst, market_grader | No |
| `grade_vs_auth` | Grader/Authenticator conflict | conflict_mediator, authentication_reviewer, grading_defender | Yes (if no consensus) |
| `high_value_deliberation` | Item value ≥ £5,000 | valuation_auditor, provenance_checker, listing_strategist | Yes |
| `rare_item_review` | Unknown/rare item | brand_historian, rarity_assessor, market_comparator | Yes |
| `defect_assessment` | Ambiguous defect vs feature | material_expert, wear_pattern_analyst, grade_impact_assessor | No |

Tribunal thresholds are modulated by AAPS personality parameters (§6): the Authenticator's high Neuroticism (0.80) lowers its escalation threshold, causing it to convene tribunals more readily on authenticity concerns, while the Copywriter's low Neuroticism (0.20) means it rarely triggers escalation.

### 11.5 Memory Integration via Formic

The swarm registers with Formic as a single entity, declaring capabilities (`product_identification`, `condition_grading`, `authenticity_verification`, `price_appraisal`, `listing_copywriting`, `quality_assurance`). At execution time:

1. **Memory Retrieval**: Relevant memories are queried from Formic (brand identification patterns, grading precedents, pricing comparables, authentication indicators) and injected into each agent's context.
2. **Learning Emission**: After successful pipeline completion, each agent's high-confidence results are stored as typed memories (FACTUAL for identification/pricing, PROCEDURAL for authentication patterns) with agent role attribution.
3. **Cross-Swarm Transfer**: Via the Gene Pool (§8), grading calibration data and authentication indicator patterns are shared across swarm instances, enabling new deployments to benefit from accumulated expertise.

### 11.6 Human Review Escalation

The pipeline computes an overall confidence score as the mean of individual agent confidences. Human review is triggered when:
*   QA Reviewer flags `requiresHumanReview`
*   Authenticator recommends `requires_expert_review`
*   Overall pipeline confidence falls below 0.70
*   Any tribunal reaches `ESCALATE_TO_HUMAN` or `NO_CONSENSUS`

---

## 12. Implementation Recommendations

### 12.1 Transport Layer

| Option | Use Case |
|--------|----------|
| gRPC | High-performance, typed messages |
| HTTP/2 | Web compatibility, streaming |
| WebSocket | Real-time, bidirectional |

### 12.2 Serialization

- **Protocol Buffers**: Recommended for production (strict typing)
- **JSON**: Development/debugging (human readable)
- **MessagePack**: Alternative binary format

### 12.3 Vector Database

For history vectors and similarity:
- pgvector (PostgreSQL extension)
- Pinecone (managed service)
- Milvus (self-hosted)
- Qdrant (Rust-based)

### 12.4 Reference Implementation Stack

```
Router: Go or Rust (performance-critical)
Agent SDK: TypeScript, Python, Go
Vector Store: pgvector or Qdrant
Message Queue: Redis Streams or NATS
Observability: OpenTelemetry
```

---

## 13. Prior Art References

This specification builds upon and distinguishes itself from:

| Prior Art | Relationship |
|-----------|--------------|
| **JSON-RPC 2.0** | LACP encapsulates JSON-RPC for tool calls |
| **gRPC** | LACP can use gRPC as transport |
| **OAuth 2.0 / OIDC** | LACP authentication compatible with OAuth |
| **MCP (Model Context Protocol)** | LACP provides security layer for MCP |
| **FIPA ACL** | Agent communication language; LACP adds security |
| **KQML** | Knowledge query language; LACP adds consensus |
| **Ant Colony Optimization** | Inspiration for pheromone-based signaling |
| **PBFT** | Byzantine fault tolerance; LACP adapts for LLM agents |
| **Commit-Reveal Schemes** | Cryptographic voting; adapted for agent consensus |

---

## 14. Data Structures

### 14.1 Context Object Schema
See `schemas/context_object.json` for the formal definition.

### 14.2 Vote Object Schema
See `schemas/vote_object.json` for the formal definition (updated with personality fields).

---

## 15. Versioning

### 15.1 Version Format

`LACP/MAJOR.MINOR`

- MAJOR: Breaking changes to protocol
- MINOR: Backward-compatible additions

### 15.2 Negotiation

Agents advertise supported versions in handshake:

```json
{
  "lacp_versions_supported": ["1.0", "1.1"],
  "lacp_version_preferred": "1.1"
}
```

---

## 16. Conclusion

LACP provides the necessary governance infrastructure for production multi-agent systems. By formalizing agent interactions into a strictly typed, layered protocol with adaptive personality-driven behaviour, LACP moves safety checks from the fragile prompt level to the robust protocol level. The Adaptive Agent Personality System introduces self-calibrating agent behaviour through psychometric personality models, while the Gene Pool enables cross-swarm knowledge transfer. Together, these mechanisms deliver enterprise-grade reliability for autonomous swarms that improve with use.

Organizations can deploy autonomous agent swarms with:
- Reduced prompt injection risk
- Prevention of sycophancy cascades
- Guaranteed objective immutability
- Auditable consensus for high-stakes decisions
- Compliance-ready PII handling
- Self-calibrating agent behaviour via OCEAN personality models
- Cross-swarm knowledge transfer via the Gene Pool

---

## 17. Publication Notice

This document is published as a defensive disclosure under the principles established by the Defensive Patent License and prior art databases. Publication establishes:

1. **Prior Art Date**: January 29, 2026 (original), February 2, 2026 (revision 2.0), February 10, 2026 (revision 3.0)
2. **Public Disclosure**: Concepts described herein are dedicated to preventing patent monopolization
3. **Implementation Rights**: Envalr and all third parties retain rights to implement these concepts
4. **No Patent Claims**: This publication does not constitute a patent application

---

**Document Classification**: Public - Defensive Publication
**Archive Copies**: Zenodo, arXiv, Internet Archive
**Contact**: legal@envalr.com
