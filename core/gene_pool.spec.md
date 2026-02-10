# LACP Core: Gene Pool (Cross-Swarm Knowledge Propagation)

## 1. Overview

The **Gene Pool** is LACP's cross-swarm knowledge propagation mechanism. It enables patterns, learnings, and strategies discovered by one swarm to benefit all others — implementing "collective intelligence" across the Swarm-of-Swarms.

Without the Gene Pool, each swarm operates in isolation: LuxeStream learns that "Chanel serial codes moved from inside the flap to inside the pocket in 2021," but that knowledge dies with the session. The Gene Pool ensures that learning persists, propagates, and evolves — so the next time any swarm encounters a Chanel item, the pattern is already available.

**Design Principles:**
*   **Asynchronous** — Swarms push and pull patterns on their own schedules. No synchronous cross-swarm calls.
*   **Fitness-Based** — Not all patterns are equal. High-accuracy, frequently-reused patterns propagate widely; stale or inaccurate patterns decay and are eventually pruned.
*   **Privacy-First** — PII is stripped before storage. Workspace isolation is enforced by default.

## 2. Architecture

### 2.1 Infrastructure

| Component | Technology | Purpose |
| :--- | :--- | :--- |
| **Database** | PostgreSQL + pgvector | Pattern storage with vector embeddings for semantic search |
| **Service** | Hono (TypeScript) | Standalone REST API service |
| **Port** | `3080` | Default service port |
| **Authentication** | API key per swarm | `GENE_POOL_API_KEY` environment variable |

### 2.2 Data Model

```sql
CREATE TABLE patterns (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    source_swarm    TEXT NOT NULL,
    domain          TEXT NOT NULL,
    pattern_type    TEXT NOT NULL,
    content         JSONB NOT NULL,
    embedding       vector(1536),
    fitness_score   FLOAT DEFAULT 0.5,
    reuse_count     INTEGER DEFAULT 0,
    last_reused_at  TIMESTAMPTZ,
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    updated_at      TIMESTAMPTZ DEFAULT NOW(),
    workspace_id    TEXT NOT NULL,
    trace_id        UUID NOT NULL
);

CREATE INDEX idx_patterns_embedding ON patterns
    USING ivfflat (embedding vector_cosine_ops);
CREATE INDEX idx_patterns_domain ON patterns (domain, fitness_score DESC);
CREATE INDEX idx_patterns_workspace ON patterns (workspace_id);
```

### 2.3 API Endpoints

| Method | Path | Description |
| :--- | :--- | :--- |
| `POST` | `/patterns` | Push a new pattern |
| `GET` | `/patterns/search` | Semantic search for relevant patterns |
| `PATCH` | `/patterns/:id/fitness` | Update fitness score |
| `GET` | `/patterns/stats` | Pattern statistics by domain/swarm |
| `DELETE` | `/patterns/:id` | Soft-delete a pattern |
| `GET` | `/health` | Service health check |

## 3. Pattern Lifecycle

Patterns flow through a 6-stage lifecycle from discovery to injection:

```
Discovery → Extraction → Fitness Scoring → Storage → Propagation → Injection
```

### 3.1 Discovery

An agent completes a task and generates learnings. Discovery happens implicitly — the Coordinator monitors agent outputs for reusable patterns.

**Examples:**
*   LuxeStream Grader learns: "Hermès Togo leather develops a distinct grain pattern when worn — this is patina, not damage."
*   Social Bot learns: "Posts with questions in the first line get 2.3x more engagement on X."
*   SafetyOS learns: "Images with text overlay in the bottom-right corner are 4x more likely to contain watermarked copyrighted content."

### 3.2 Extraction

The Coordinator extracts the reusable pattern from the agent's raw output. Extraction strips context-specific details and generalises the learning.

**Extraction Rules:**
*   Remove item-specific identifiers (SKUs, URLs, customer data).
*   Preserve the generalised principle.
*   Tag with domain metadata (brand, category, platform).
*   Generate a vector embedding for semantic retrieval.

**Pattern Types:**
| Type | Description | Example |
| :--- | :--- | :--- |
| `BRAND_QUIRK` | Brand-specific manufacturing detail | "Louis Vuitton date codes use factory codes + week/year" |
| `DEFECT_SIGNATURE` | Visual pattern indicating a specific defect | "Cracking along fold lines on Saffiano leather indicates age, not misuse" |
| `PRICING_CORRECTION` | Market pricing adjustment | "Vintage Chanel flap bags command 20% premium over current retail" |
| `ENGAGEMENT_PATTERN` | Content/interaction pattern | "Luxury item posts perform best between 7-9pm GMT" |
| `SAFETY_MARKER` | Content safety signal | "Counterfeit Gucci uses incorrect font spacing on the 'Made in Italy' tag" |
| `GENERAL` | Unclassified reusable learning | Catch-all for patterns that do not fit other types |

### 3.3 Fitness Scoring

Every pattern has a **fitness score** (0.0 - 1.0) that determines its propagation priority and injection likelihood.

**Initial Score:** `0.5` (neutral). Patterns must prove themselves before reaching high fitness.

**Score Adjustments:**

| Signal | Direction | Magnitude | Description |
| :--- | :--- | :--- | :--- |
| Human validation | Up | +0.2 | A human expert confirms the pattern is accurate |
| Successful reuse | Up | +0.05 | Another swarm uses the pattern and achieves good outcome |
| Human rejection | Down | -0.3 | A human expert marks the pattern as incorrect |
| Contradiction | Down | -0.15 | Another pattern with higher fitness contradicts this one |
| Time decay | Down | See 4.3 | Fitness decays if the pattern is not reused |

**Thresholds:**

| Range | Classification | Behaviour |
| :--- | :--- | :--- |
| 0.8 - 1.0 | **Gold** | Injected by default into all relevant agent contexts |
| 0.5 - 0.79 | **Silver** | Injected on semantic match (cosine similarity > 0.7) |
| 0.2 - 0.49 | **Bronze** | Available on explicit pull only; not auto-injected |
| 0.0 - 0.19 | **Dormant** | Excluded from search results; candidate for pruning |

### 3.4 Storage

Patterns are stored in PostgreSQL with pgvector embeddings for semantic retrieval.

**Storage Payload:**
```json
{
  "source_swarm": "luxestream",
  "domain": "authentication",
  "pattern_type": "BRAND_QUIRK",
  "content": {
    "summary": "Chanel serial codes moved from inside the flap to inside the pocket in 2021",
    "detail": "For Classic Flap bags manufactured after March 2021, the serial sticker is located inside the interior pocket rather than behind the flap. Both locations are authentic.",
    "brands": ["Chanel"],
    "categories": ["handbags"],
    "confidence": 0.92
  },
  "workspace_id": "ws-prod-001",
  "trace_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

### 3.5 Propagation

Propagation is **asynchronous** and follows a push/pull model:

*   **Push:** After a mission completes, the Coordinator pushes extracted patterns to the Gene Pool. This is fire-and-forget — the mission does not wait for acknowledgement.
*   **Pull:** At session start (or mission start), swarms pull relevant patterns by performing a semantic search against their current context. Pulled patterns are cached locally for the duration of the session.

**Propagation is not broadcast.** Patterns are stored centrally and retrieved on demand. There is no pub/sub fan-out — this keeps the system simple and avoids flooding swarms with irrelevant patterns.

### 3.6 Injection

Retrieved patterns are injected into agent context via the `formicContext` field in the LACP Context Object:

```json
{
  "formicContext": {
    "genePoolPatterns": [
      {
        "id": "pat-a1b2c3",
        "type": "BRAND_QUIRK",
        "summary": "Chanel serial codes moved from inside the flap to inside the pocket in 2021",
        "fitness": 0.88,
        "source_swarm": "luxestream",
        "relevance": 0.94
      }
    ]
  }
}
```

*   **Relevance Score:** Cosine similarity between the pattern embedding and the current mission context embedding. Only patterns above the relevance threshold for their fitness tier are injected.
*   **Max Injection Count:** 10 patterns per agent context (to prevent prompt bloat).
*   **Ordering:** Patterns are ordered by `fitness * relevance` descending.

## 4. Cross-Swarm Semantics

### 4.1 Tagging

Every pattern is tagged with:
*   **`source_swarm`** — The swarm that discovered the pattern (e.g., `luxestream`, `social-bot`, `safety-os`).
*   **`domain`** — The knowledge domain (e.g., `authentication`, `grading`, `engagement`, `safety`).
*   **`confidence`** — The source agent's confidence at time of extraction (0.0 - 1.0).

### 4.2 Domain Relevance Filter

Receiving swarms apply a **domain relevance filter** before injecting patterns into agent context. This prevents irrelevant patterns from polluting agent prompts.

**Filter Logic:**
1.  Retrieve candidate patterns via semantic search (cosine similarity > 0.5).
2.  Apply domain affinity matrix (see below) to weight relevance.
3.  Discard patterns below the effective relevance threshold.

**Domain Affinity Matrix (Example):**

| Receiving Swarm | High Affinity Domains | Medium Affinity | Low Affinity |
| :--- | :--- | :--- | :--- |
| LuxeStream | `grading`, `authentication`, `pricing` | `safety`, `engagement` | `social`, `code` |
| Social Bot | `engagement`, `social`, `content` | `safety`, `brand` | `grading`, `pricing` |
| SafetyOS | `safety`, `authentication`, `content` | `engagement` | `grading`, `pricing` |

### 4.3 Fitness Decay

Patterns lose fitness over time if not reused. This prevents the Gene Pool from accumulating stale knowledge.

**Decay Function:**

$$
f(t) = f_0 \times 2^{-t/\tau}
$$

Where:
*   $f_0$ = fitness score at last reuse
*   $t$ = days since last reuse
*   $\tau$ = half-life (default: **30 days**)

**Example:** A pattern with fitness 0.8, unused for 30 days, decays to 0.4. After 60 days without reuse, it drops to 0.2 (Dormant tier) and is excluded from search results.

**Decay is paused** when a pattern is reused. Each reuse resets the decay clock.

### 4.4 Contradiction Resolution

When two swarms produce conflicting patterns (e.g., Swarm A says "Chanel moved serial codes in 2021" and Swarm B says "Chanel moved serial codes in 2020"):

1.  **Detection:** Contradictions are detected via high semantic similarity (> 0.9) combined with content divergence (detected by embedding the `detail` fields and checking for opposing claims).
2.  **Resolution:** The pattern with the higher fitness score wins. The losing pattern receives a fitness penalty of -0.15.
3.  **Flagging:** Both patterns are flagged for human review. A `CONTRADICTION` event is logged with both pattern IDs and the trace IDs of the originating missions.
4.  **Human Override:** A human can manually resolve the contradiction by validating one pattern and rejecting the other.

## 5. LACP Transport

Gene Pool messages use the standard LACP transport layer (see `transport.spec.md`) with the following conventions:

### 5.1 Message Format

```json
{
  "headers": {
    "x-lacp-ver": "1.0",
    "x-trace-id": "550e8400-e29b-41d4-a716-446655440000",
    "x-sender-id": "gene-pool-service",
    "x-ttl": 1
  },
  "body": {
    "mission_id": "gene-pool-sync-luxestream-001",
    "payload": {
      "type": "PATTERN_PUSH",
      "patterns": [ ... ]
    }
  }
}
```

### 5.2 Payload Types

| Type | Direction | Description |
| :--- | :--- | :--- |
| `PATTERN_PUSH` | Swarm -> Gene Pool | Push new patterns after mission completion |
| `PATTERN_PULL` | Swarm -> Gene Pool | Request relevant patterns for current context |
| `FITNESS_UPDATE` | Swarm -> Gene Pool | Report pattern reuse outcome (success/failure) |

### 5.3 Mission ID Convention

Gene Pool sync operations use a predictable mission ID format:

```
gene-pool-sync-{swarm_id}
```

This allows easy filtering and monitoring of Gene Pool traffic in logs and dashboards.

### 5.4 Rate Limiting

Gene Pool operations are rate limited to prevent flooding:

*   **Limit:** 10 RPM (Requests Per Minute) per swarm.
*   **Mechanism:** Token Bucket (consistent with LACP Layer 1 rate limiting).
*   **Burst:** Up to 20 requests allowed in a burst window, then throttled.
*   **Headers:** Standard `X-RateLimit-*` headers returned (see `transport.spec.md` Section 2).

**Rationale:** Gene Pool operations are background/async. There is no user-facing latency requirement, so aggressive rate limiting is acceptable to protect the database from write storms during high-throughput batch processing.

## 6. Privacy & Security

### 6.1 PII Scanning

All patterns are scanned for PII before storage. This applies the same PII Redaction rules defined in LACP Layer 1.

**Scanned Fields:**
*   `content.summary`
*   `content.detail`
*   All string values within `content`

**Redaction Rules:**
*   Email addresses -> `[REDACTED_EMAIL]`
*   Phone numbers -> `[REDACTED_PHONE]`
*   Names (when detected with high confidence) -> `[REDACTED_NAME]`
*   Addresses -> `[REDACTED_ADDRESS]`
*   Financial data (card numbers, account numbers) -> `[REDACTED_FINANCIAL]`

Patterns that contain PII after redaction would be meaningless (e.g., "Contact [REDACTED_NAME] at [REDACTED_EMAIL] for authentication") are rejected and not stored.

### 6.2 Workspace Isolation

**Default:** Patterns only propagate within the same `workspace_id`. A pattern created by LuxeStream in workspace `ws-prod-001` is invisible to swarms in `ws-prod-002`.

**Explicit Sharing:** Workspace administrators can mark specific patterns as `shared`, making them available across workspaces within the same organisation. Shared patterns are read-only in receiving workspaces — they cannot be modified or deleted by non-owning swarms.

**Cross-Organisation:** Not supported. Patterns never cross organisation boundaries.

### 6.3 Audit Trail

All Gene Pool operations are logged with full traceability:

| Field | Description |
| :--- | :--- |
| `trace_id` | UUID linking to the originating LACP mission |
| `action` | `PUSH`, `PULL`, `FITNESS_UPDATE`, `PRUNE`, `CONTRADICTION` |
| `actor` | Swarm ID or `system` (for automated operations like decay) |
| `timestamp` | ISO 8601 timestamp |
| `workspace_id` | Workspace scope |
| `pattern_id` | Affected pattern(s) |

**Retention:** Audit logs are retained for 90 days by default, configurable per workspace.

**Compliance:** The audit trail satisfies the traceability requirements of LACP Layer 1 (see `transport.spec.md`) and provides the data needed for the LACP Disclosure Bundle (see `../LACP_Disclosure_Document.md`).
