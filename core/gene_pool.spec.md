# LACP Core: Gene Pool (Cross-Swarm Knowledge Propagation)

## 1. Overview

The **Gene Pool** is LACP's cross-swarm knowledge propagation mechanism. It enables patterns, learnings, and strategies discovered by one swarm to benefit all others — implementing "collective intelligence" across the Swarm-of-Swarms.

Without the Gene Pool, each swarm operates in isolation: learnings from one domain die with the session. The Gene Pool ensures that learning persists, propagates, and evolves — so the next time any swarm encounters a similar situation, the pattern is already available.

**Design Principles:**
*   **Asynchronous** — Swarms push and pull patterns on their own schedules. No synchronous cross-swarm calls.
*   **Fitness-Based** — Not all patterns are equal. High-accuracy, frequently-reused patterns propagate widely; stale or inaccurate patterns decay and are eventually pruned.
*   **Privacy-First** — PII is stripped before storage. Workspace isolation is enforced by default.

## 2. Architecture

### 2.1 Infrastructure

The Gene Pool is implemented as a standalone service with:
*   A relational database with vector extension support for semantic search.
*   A REST API service for pattern CRUD and search.
*   API key authentication per swarm.

### 2.2 Data Model

Patterns are stored with the following fields:
*   `id` — Unique identifier
*   `source_swarm` — The swarm that discovered the pattern
*   `domain` — Knowledge domain
*   `pattern_type` — Classification (see Section 3.2)
*   `content` — JSON payload with summary, detail, and metadata
*   `embedding` — Vector embedding for semantic retrieval
*   `fitness_score` — Quality score (0.0 - 1.0)
*   `reuse_count` — Number of times the pattern has been successfully reused
*   `workspace_id` — Workspace isolation boundary
*   `trace_id` — Traceability link to the originating mission

### 2.3 API

| Method | Path | Description |
| :--- | :--- | :--- |
| `POST` | `/patterns` | Push a new pattern |
| `GET` | `/patterns/search` | Semantic search for relevant patterns |
| `PATCH` | `/patterns/:id/fitness` | Update fitness score |
| `GET` | `/patterns/stats` | Pattern statistics |
| `DELETE` | `/patterns/:id` | Soft-delete a pattern |
| `GET` | `/health` | Service health check |

## 3. Pattern Lifecycle

Patterns flow through a 6-stage lifecycle from discovery to injection:

```
Discovery → Extraction → Fitness Scoring → Storage → Propagation → Injection
```

### 3.1 Discovery

An agent completes a task and generates learnings. Discovery happens implicitly — the Coordinator monitors agent outputs for reusable patterns.

### 3.2 Extraction

The Coordinator extracts the reusable pattern from the agent's raw output. Extraction strips context-specific details and generalises the learning.

**Extraction Rules:**
*   Remove item-specific identifiers (SKUs, URLs, customer data).
*   Preserve the generalised principle.
*   Tag with domain metadata.
*   Generate a vector embedding for semantic retrieval.

**Pattern Types:**
| Type | Description |
| :--- | :--- |
| `BRAND_QUIRK` | Domain-specific detail or exception |
| `DEFECT_SIGNATURE` | Pattern indicating a specific defect or anomaly |
| `PRICING_CORRECTION` | Market pricing adjustment |
| `ENGAGEMENT_PATTERN` | Content/interaction pattern |
| `SAFETY_MARKER` | Content safety signal |
| `GENERAL` | Unclassified reusable learning |

### 3.3 Fitness Scoring

Every pattern has a **fitness score** (0.0 - 1.0) that determines its propagation priority and injection likelihood.

**Initial Score:** `0.5` (neutral). Patterns must prove themselves before reaching high fitness.

**Score Adjustments:**

| Signal | Direction | Description |
| :--- | :--- | :--- |
| Human validation | Up | A human expert confirms the pattern is accurate |
| Successful reuse | Up | Another swarm uses the pattern and achieves good outcome |
| Human rejection | Down | A human expert marks the pattern as incorrect |
| Contradiction | Down | Another pattern with higher fitness contradicts this one |
| Time decay | Down | Fitness decays if the pattern is not reused |

**Fitness Tiers:**

| Tier | Behaviour |
| :--- | :--- |
| **Gold** | Injected by default into all relevant agent contexts |
| **Silver** | Injected on semantic match above relevance threshold |
| **Bronze** | Available on explicit pull only; not auto-injected |
| **Dormant** | Excluded from search results; candidate for pruning |

### 3.4 Propagation

Propagation is **asynchronous** and follows a push/pull model:

*   **Push:** After a mission completes, the Coordinator pushes extracted patterns to the Gene Pool. This is fire-and-forget.
*   **Pull:** At session start, swarms pull relevant patterns by performing a semantic search against their current context. Pulled patterns are cached locally for the session duration.

Propagation is not broadcast. Patterns are stored centrally and retrieved on demand.

### 3.5 Injection

Retrieved patterns are injected into agent context via the `formicContext` field in the LACP Context Object:

```json
{
  "formicContext": {
    "genePoolPatterns": [
      {
        "id": "pat-xxx",
        "type": "GENERAL",
        "summary": "Example pattern summary",
        "fitness": 0.0,
        "source_swarm": "example",
        "relevance": 0.0
      }
    ]
  }
}
```

*   **Relevance Score:** Cosine similarity between the pattern embedding and the current mission context embedding.
*   **Max Injection Count:** Limited per agent context to prevent prompt bloat.
*   **Ordering:** Patterns are ordered by `fitness * relevance` descending.

## 4. Cross-Swarm Semantics

### 4.1 Tagging

Every pattern is tagged with:
*   **`source_swarm`** — The swarm that discovered the pattern.
*   **`domain`** — The knowledge domain.
*   **`confidence`** — The source agent's confidence at time of extraction.

### 4.2 Domain Relevance Filter

Receiving swarms apply a **domain relevance filter** before injecting patterns. This prevents irrelevant patterns from polluting agent prompts.

**Filter Logic:**
1.  Retrieve candidate patterns via semantic search.
2.  Apply domain affinity weights to adjust relevance scores.
3.  Discard patterns below the effective relevance threshold.

### 4.3 Fitness Decay

Patterns lose fitness over time if not reused, preventing the Gene Pool from accumulating stale knowledge.

**Decay Function:** Exponential half-life decay. Each reuse resets the decay clock.

### 4.4 Contradiction Resolution

When two swarms produce conflicting patterns:

1.  **Detection:** High semantic similarity combined with content divergence.
2.  **Resolution:** The pattern with the higher fitness score wins. The losing pattern receives a fitness penalty.
3.  **Flagging:** Both patterns are flagged for human review.
4.  **Human Override:** A human can manually resolve the contradiction.

## 5. LACP Transport

Gene Pool messages use the standard LACP transport layer (see `transport.spec.md`).

### 5.1 Payload Types

| Type | Direction | Description |
| :--- | :--- | :--- |
| `PATTERN_PUSH` | Swarm -> Gene Pool | Push new patterns after mission completion |
| `PATTERN_PULL` | Swarm -> Gene Pool | Request relevant patterns for current context |
| `FITNESS_UPDATE` | Swarm -> Gene Pool | Report pattern reuse outcome |

### 5.2 Rate Limiting

Gene Pool operations are rate limited to prevent flooding:
*   **Mechanism:** Token Bucket (consistent with LACP Layer 1 rate limiting).
*   Gene Pool operations are background/async, so aggressive rate limiting is acceptable.

## 6. Privacy & Security

### 6.1 PII Scanning

All patterns are scanned for PII before storage, applying the same PII Redaction rules defined in LACP Layer 1. Patterns that would be meaningless after redaction are rejected.

### 6.2 Workspace Isolation

**Default:** Patterns only propagate within the same `workspace_id`.

**Explicit Sharing:** Workspace administrators can mark specific patterns as `shared`, making them available across workspaces within the same organisation. Shared patterns are read-only in receiving workspaces.

**Cross-Organisation:** Not supported.

### 6.3 Audit Trail

All Gene Pool operations are logged with full traceability (trace_id, action, actor, timestamp, workspace_id, pattern_id). The audit trail satisfies LACP Layer 1 traceability requirements.
