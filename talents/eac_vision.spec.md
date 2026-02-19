# LACP Talent: EAC-Vision (Governance Documentation Pipeline)

## 1. Overview

**EAC-Vision** is a governance documentation swarm that assesses documentation maturity, identifies compliance gaps, and enforces ADR (Architecture Decision Record) governance across GitHub repositories. It operates 3 specialised agents in a sequential pipeline: an Assessor ingests repositories and scores them against a 6-level maturity model (L0-L5), a Planner generates gap analysis and prioritised roadmaps, and an Auditor validates compliance against organisational policies.

The swarm targets engineering organisations that want to systematically improve their documentation practices. It ingests code repositories, extracts existing documentation artefacts (README, ADRs, API docs, runbooks), evaluates them against a maturity rubric, and produces actionable improvement plans with measurable milestones.

## 2. Swarm Architecture

EAC-Vision operates a **3-stage sequential pipeline** with 3 specialised agents. Each stage depends on the output of the previous stage.

```
Stage 1: ASSESSOR (Analyst)
    |   GitHub repo ingestion via Octokit
    |   Documentation extraction and classification
    |   Maturity scoring against L0-L5 model
    |
    v
Stage 2: PLANNER (Architect)
    |   Gap analysis: current state vs north-star target
    |   Prioritised roadmap generation
    |   Effort estimation and dependency mapping
    |
    v
Stage 3: AUDITOR (Reviewer)
        ADR governance validation (MADR format)
        Policy enforcement checks
        Cross-repo consistency analysis
```

**Design Rationale:**
*   The pipeline is strictly sequential — the Planner cannot generate a roadmap without the Assessor's maturity scores, and the Auditor cannot validate compliance without the Planner's target state definition.
*   Each stage produces a structured output that becomes immutable context for downstream agents, ensuring full traceability from raw repository data to compliance recommendations.
*   The 6-level maturity model (L0: Absent, L1: Ad-hoc, L2: Documented, L3: Standardised, L4: Measured, L5: Optimised) provides a universal rubric applicable across any technology stack.

## 3. Agent Roles

| Agent | LACP Role | Archetype | Model | Trust Level | Function |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Assessor** | `ANALYST` | Examiner | `claude-sonnet-4-5` | `INTERNAL` | Ingests GitHub repos, extracts ADRs, evaluates documentation against 6-level maturity model |
| **Planner** | `ARCHITECT` | Strategist | `claude-sonnet-4-5` | `INTERNAL` | Compares current state against north-star targets, generates actionable roadmaps |
| **Auditor** | `REVIEWER` | Inspector | `claude-sonnet-4-5` | `INTERNAL` | Validates ADR governance (MADR format), policy enforcement, cross-repo consistency |

**Model Selection Notes:**
*   All three agents use `claude-sonnet-4-5` because each task requires structured reasoning over large document sets. The Assessor must classify and score diverse documentation formats, the Planner must synthesise gaps into prioritised plans, and the Auditor must enforce precise governance rules.
*   Local embeddings via Ollama supplement the LLM agents for semantic search and document similarity, but the core reasoning pipeline remains LLM-driven.

## 4. LACP Interface

### 4.1 Role Mapping

| Domain Role | LACP Role | Justification |
| :--- | :--- | :--- |
| `ASSESSOR` | `ANALYST` | Analyses repository documentation to produce structured maturity assessments |
| `PLANNER` | `ARCHITECT` | Architects the improvement roadmap — designs the path from current to target state |
| `AUDITOR` | `REVIEWER` | Reviews outputs and validates compliance against governance policies |

### 4.2 Trust Level

All agents operate at `INTERNAL` trust level. GitHub repository access is mediated through Octokit with scoped tokens — agents never receive raw credentials. Repository content is ingested into the local PostgreSQL store and processed in-place.

### 4.3 Context Object

```json
{
  "mission_id": "eac-assess-{org}-{timestamp}",
  "objective": "Assess documentation maturity and generate improvement roadmap",
  "payload": {
    "repositories": [
      "github.com/org/service-api",
      "github.com/org/platform-core"
    ],
    "targetMaturity": "L3",
    "policySet": "enterprise-default",
    "adrFormat": "MADR",
    "constraints": {
      "maxRepos": 50,
      "includeArchived": false
    },
    "formicContext": {
      "memories": [],
      "genePoolPatterns": [],
      "previousAssessments": []
    }
  }
}
```

*   **`repositories[]`** — GitHub repository URLs to assess. Supports org-wide scanning when a single org URL is provided.
*   **`targetMaturity`** — North-star maturity level (L0-L5) used by the Planner for gap analysis.
*   **`policySet`** — Governance policy bundle the Auditor validates against.
*   **`adrFormat`** — Expected ADR format. Currently supports `MADR` (Markdown Any Decision Record).
*   **`previousAssessments[]`** — Injected by the Coordinator. Historical assessments enable maturity trend tracking.

### 4.4 Feature Flag

```
USE_EAC_PIPELINE=true
```

*   **`true`** — Activates the full 3-agent pipeline.
*   **`false`** — Falls back to single-model assessment with simplified scoring.
*   **Auto-fallback:** If GitHub API rate limits are hit, the Assessor operates on cached repository snapshots from the last successful ingestion.

## 5. Tribunal Integration

EAC-Vision defines **3 domain-specific conditions** that trigger a Tribunal review. When triggered, the pipeline pauses and a Tribunal is convened using the standard Commit-Reveal flow (see `../consensus/tribunal.spec.md`).

### 5.1 Trigger Conditions

| # | Condition | Trigger Rule | Jury Composition |
| :--- | :--- | :--- | :--- |
| 1 | **Maturity regression** | Repository maturity drops below previous assessment level | `ASSESSOR` + `AUDITOR` + `PLANNER` |
| 2 | **Compliance gap** | Mandatory ADR missing for an identified architectural decision | `AUDITOR` + `ASSESSOR` + `PLANNER` |
| 3 | **Stale documentation** | Critical documentation not updated > 90 days | `AUDITOR` + `PLANNER` + `ASSESSOR` |

### 5.2 Tribunal Mechanics

*   **Voting:** Personality-weighted blind voting with sycophancy detection (see `../consensus/sycophancy.spec.md`).
*   **Personality Weighting:** The Auditor's high Conscientiousness and low Agreeableness give it the highest effective vote weight, reflecting its role as the governance authority.
*   **Safety Default:** If the Tribunal cannot reach consensus, the assessment is flagged as "requires human review" with all agent reasoning traces attached.
*   **Escalation:** Assessments that fail Tribunal twice are escalated to the engineering governance board with full context and dissenting opinions.

## 6. Personality System (AAPS)

Each EAC-Vision agent has an **OCEAN personality profile** managed by the Agent Adaptive Personality System (AAPS). Refer to `../consensus/personality.spec.md` for the full specification.

### 6.1 Default Archetypes

Temperature formula: `temp = clamp(O * 0.6 + E * 0.2 - C * 0.15 - N * 0.1, 0.0, 1.0)`

| Agent | Archetype | O | C | E | A | N | Derived Temp |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Assessor | Examiner | 0.60 | 0.80 | 0.45 | 0.50 | 0.45 | ~0.29 |
| Planner | Strategist | 0.75 | 0.70 | 0.60 | 0.65 | 0.30 | ~0.44 |
| Auditor | Inspector | 0.25 | 0.95 | 0.30 | 0.20 | 0.75 | ~0.00 |

**Calibration Note:** The Auditor has the lowest derived temperature (~0.00, clamped from a slightly negative raw value), reflecting its strict, deterministic compliance-checking role. The Planner has the highest temperature (~0.44), enabling it to generate creative but practical improvement roadmaps. The Assessor sits in between (~0.29), balancing analytical rigour with enough flexibility to handle novel documentation patterns.

### 6.2 Personality Evolution

Personality profiles evolve based on three feedback signals:

1.  **Human Overrides** — When a governance lead overrides an assessment or rejects a roadmap item, the responsible agent's profile is nudged (e.g., an Assessor that consistently over-scores maturity has Conscientiousness increased).
2.  **Tribunal Outcomes** — Agents whose votes align with the final verdict receive a small Conscientiousness boost.
3.  **Adoption Metrics** — When roadmap recommendations are implemented and result in measurable maturity improvement, the Planner's profile is reinforced. Recommendations that are repeatedly ignored trigger an Openness reduction (making future suggestions more conservative and practical).

## 7. Tech Stack

| Component | Technology | Purpose |
| :--- | :--- | :--- |
| CLI | TypeScript | Primary interface for running assessments |
| ORM | Prisma | Database schema management and query layer |
| Primary DB | PostgreSQL + pgvector | Structured assessment data and semantic document search |
| Embeddings | Ollama (local) | Document embeddings for semantic similarity and clustering |
| LLM | Claude API | Classification, distillation, and structured reasoning |
| GitHub | Octokit | Repository ingestion, ADR extraction, and file tree traversal |

## 8. Cross-Swarm Interactions

| Pattern Flow | Source | Destination | Example |
| :--- | :--- | :--- | :--- |
| Governance patterns | EAC-Vision | **Gene Pool** | Documentation maturity rubrics and ADR templates shared as reusable patterns |
| Assessment methodology | EAC-Vision | **Knowledge Archive** | Maturity assessment techniques and scoring algorithms |
| Code quality patterns | Formic (QA_ENGINEER) | **EAC-Vision** | Code review findings inform documentation gap detection |
| Safety policies | SafetyOS | **EAC-Vision** | Security documentation requirements injected into Auditor policy sets |
| Documentation standards | EAC-Vision | **Formic Core** | Standardised documentation templates propagated to all swarms |

**Propagation Mechanism:** All cross-swarm interactions flow through the Gene Pool's async push/pull protocol. No direct swarm-to-swarm communication occurs — the Gene Pool acts as the sole intermediary, ensuring workspace isolation and audit traceability.
