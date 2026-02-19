# LACP Talent: Formic (The Orchestrator Swarm)

## 1. Overview

**Formic** is the core AI agent orchestration platform — the colony substrate upon which all other swarms execute. It provides mission decomposition, agent lifecycle management, constitutional governance, and cross-swarm coordination for the entire ecosystem.

Unlike product-level swarms (Provenance, TalentOS, DVS) that solve specific domain problems, Formic is the **meta-swarm**: it manages the execution of other swarms, enforces non-functional requirements (NFRs) as constitutional constraints, and propagates learnings across the ecosystem via the Gene Pool. Every mission in every product swarm ultimately flows through Formic's coordination layer.

Formic operates 18 specialised agent roles, each mapped to an ant colony archetype that reflects its function within the collective.

## 2. Swarm Architecture

Formic operates a **mission pipeline** with parallel execution paths. The pipeline decomposes high-level objectives into executable tasks, routes them to specialist agents, and enforces quality and compliance gates before delivery.

```
Stage 1: VISIONARY (gyne/Queen)
    │   Strategic analysis, objective decomposition
    │
    ▼
Stage 2: PROJECT_MANAGER (pheromone)
    │   Mission planning, task graph construction
    │
    ├──────────────────────────────────────────────────────┐
    ▼                                                      ▼
Stage 3a: Design Path                           Stage 3b: Research Path
    │                                               │
    ARCHITECT (mason)                               RESEARCHER (scout)
    │   System design, architecture decisions       │   Investigation, knowledge discovery
    │                                               │
    ├→ DESIGNER (morph)                             └→ BUILDER (ergate)
    │   UI/UX design, prototyping                      Implementation from research
    │
    └→ BUILDER (ergate)
       Implementation from design
       │
       ▼
    QA_ENGINEER (inspector)
       Testing, validation
       │
    ├──────────────────────────────────────────────────────┘
    ▼
Stage 4: Compliance Gate
    │
    ├→ AUDITOR (sentinel) ─── NFR validation, compliance review
    ├→ SECOPS (guardian) ──── Security scanning, vulnerability detection
    ├→ DEVSECOPS (cleaner) ── Pipeline security, dependency audit
    │
    ▼
Stage 5: Quality Gate
    │
    CHIEF_OF_STAFF ─── Final review, delivery coordination
    │
    ▼
Stage 6: Operations
    │
    ├→ DEVOPS (carrier) ──────── Deployment, infrastructure
    ├→ SERVICE_MANAGER (trophallaxis) ── Monitoring, incident response
    └→ FINOPS (replete) ─────── Cost tracking, budget enforcement
```

**Design Rationale:**
*   Stages 3a and 3b execute in parallel — design-driven and research-driven work streams are independent and converge only at the compliance gate.
*   The compliance gate (Stage 4) runs AUDITOR, SECOPS, and DEVSECOPS concurrently. Each performs independent checks; all must pass for the mission to proceed.
*   Pheromone coordination enables dynamic re-routing: if SECOPS emits a `SECURITY_AUDIT` pheromone mid-pipeline, all downstream agents pause until the audit resolves.
*   The pipeline supports stigmergic coordination — agents detect codebase state changes (test failures, lint errors, security warnings) and adjust behaviour without explicit instruction.

## 3. Agent Roles

| Agent | LACP Role | Archetype | Ant Analogy | Trust Level | Function |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Visionary** | `VISIONARY` | Gyne | Queen — sets colony direction | `INTERNAL` | Strategic analysis, objective decomposition, produces plans and high-level analysis |
| **Project Manager** | `COORDINATOR` | Pheromone | Chemical signals — coordinates without direct contact | `INTERNAL` | Mission planning, task decomposition, dependency management, progress tracking |
| **Business Analyst** | `ANALYST` | Forager | Searches for resources — discovers requirements | `INTERNAL` | Requirements gathering, stakeholder analysis, documentation, acceptance criteria |
| **Architect** | `ARCHITECT` | Mason | Builds structural foundations | `INTERNAL` | System design, architecture decisions, technical specifications, NFR definition |
| **Builder** | `BUILDER` | Ergate | Worker — the colony's labour force | `INTERNAL` | Code implementation, unit testing, feature development, bug fixes |
| **QA Engineer** | `TESTER` | Inspector | Quality control at colony junctions | `INTERNAL` | Test planning, test execution, coverage analysis, regression testing |
| **Auditor** | `REVIEWER` | Sentinel | Guards the colony perimeter | `INTERNAL` | NFR compliance validation, code review, standards enforcement |
| **DevOps** | `DEVOPS` | Carrier | Transports resources through tunnels | `INTERNAL` | CI/CD pipelines, infrastructure provisioning, deployment automation |
| **Service Manager** | `SERVICE_MANAGER` | Trophallaxis | Food sharing — monitors colony health | `INTERNAL` | Service monitoring, incident response, SLA tracking, operational health |
| **FinOps** | `TREASURER` | Replete | Stores food reserves — manages resources | `INTERNAL` | Compute cost tracking, budget enforcement, cost optimisation recommendations |
| **SecOps** | `SECURITY` | Guardian | Colony defence specialist | `INTERNAL` | Threat detection, vulnerability scanning, secret exposure prevention, incident response |
| **DevSecOps** | `DEVSECOPS` | Cleaner | Keeps tunnels free of parasites | `INTERNAL` | Pipeline security, dependency auditing, SAST/DAST integration, supply chain security |
| **HR Manager** | `COACH` | Nurse | Tends to larvae — develops agents | `INTERNAL` | Agent capability assessment, training data curation, performance evaluation |
| **Designer** | `DESIGNER` | Morph | Shape-shifting — adapts form to function | `INTERNAL` | UI/UX design, wireframing, prototyping, design system maintenance |
| **Debugger** | `DEBUGGER` | Soldier | Confronts threats head-on | `INTERNAL` | Root cause analysis, stack trace investigation, reproduction case construction |
| **Researcher** | `EXPLORER` | Scout | Explores unknown territory | `INTERNAL` | Technology research, competitive analysis, knowledge discovery, PoC development |
| **Discovery** | `DOCUMENTER` | Antenna | Senses environment — detects patterns | `INTERNAL` | Codebase exploration, pattern recognition, documentation generation, API mapping |
| **Chief of Staff** | `CHIEF_OF_STAFF` | — | Colony's executive coordinator | `INTERNAL` | Cross-team coordination, delegation, progress monitoring, delivery assurance |

**Model Selection Notes:**
*   Agent model assignment is dynamic — the Coordinator selects models based on task complexity, budget constraints, and the `FINOPS` agent's cost recommendations.
*   High-reasoning tasks (architecture, security review, debugging) default to `claude-sonnet-4-5` or `gemini-2.5-pro`.
*   Structured extraction and generation tasks default to `gemini-2.0-flash` or `claude-haiku` for cost efficiency.
*   The `VISIONARY` role always uses a frontier model, as strategic decisions have the highest downstream impact.

## 4. LACP Interface

### 4.1 Role Mapping

Formic's 18 internal roles map to standard LACP roles. Where multiple agents share an LACP role, the Coordinator disambiguates by capability tags.

| Domain Role | LACP Role | Justification |
| :--- | :--- | :--- |
| `VISIONARY` | `VISIONARY` | Sets strategic direction — unique LACP role for colony leadership |
| `PROJECT_MANAGER` | `COORDINATOR` | Orchestrates task flow and agent assignment |
| `BUSINESS_ANALYST` | `ANALYST` | Discovers and formalises requirements |
| `ARCHITECT` | `ARCHITECT` | Designs system structure and technical plans |
| `BUILDER` | `BUILDER` | Produces implementation artefacts (code, configs) |
| `QA_ENGINEER` | `TESTER` | Validates correctness via test execution |
| `AUDITOR` | `REVIEWER` | Reviews outputs against NFR constraints |
| `DEVOPS` | `DEVOPS` | Manages deployment and infrastructure lifecycle |
| `SERVICE_MANAGER` | `SERVICE_MANAGER` | Monitors and maintains operational health |
| `FINOPS` | `TREASURER` | Manages budget and cost allocation |
| `SECOPS` | `SECURITY` | Detects and responds to security threats |
| `DEVSECOPS` | `DEVSECOPS` | Secures the build/deploy pipeline |
| `HR_MANAGER` | `COACH` | Develops agent capabilities and evaluates performance |
| `DESIGNER` | `DESIGNER` | Produces visual and interaction design artefacts |
| `DEBUGGER` | `DEBUGGER` | Investigates and resolves defects |
| `RESEARCHER` | `EXPLORER` | Explores unknown problem spaces and technologies |
| `DISCOVERY` | `DOCUMENTER` | Documents codebases, APIs, and discovered patterns |
| `CHIEF_OF_STAFF` | `CHIEF_OF_STAFF` | Executive coordination and delivery assurance |

### 4.2 Trust Level

All agents operate at `INTERNAL` trust level by default. Product swarms that integrate via LACP may expose specific agents at `PARTNER` trust level for cross-swarm interactions, but the core Formic agents never accept external input directly.

### 4.3 Context Object

The mission payload conforms to the standard LACP Context Object (see `../core/context.spec.md`) with the following orchestration-specific fields:

```json
{
  "mission_id": "formic-mission-{uuid}",
  "objective": "High-level mission objective in natural language",
  "payload": {
    "missionType": "feature | bugfix | refactor | research | security-audit",
    "workspace": "workspace-uuid",
    "constraints": {
      "budgetCeiling": 5.00,
      "timeoutMinutes": 120,
      "nfrProfile": "default | strict | relaxed"
    },
    "pheromones": [],
    "taskGraph": {
      "nodes": [],
      "edges": [],
      "parallelGroups": []
    },
    "formicContext": {
      "memories": [],
      "genePoolPatterns": [],
      "constitutionalConstraints": []
    }
  }
}
```

*   **`missionType`** — Determines the default pipeline path and agent selection heuristic.
*   **`constraints.budgetCeiling`** — Maximum compute spend in USD. FINOPS monitors and triggers a Tribunal if exceeded.
*   **`constraints.nfrProfile`** — NFR strictness level. `strict` enables all compliance gates; `relaxed` skips non-critical audits for speed.
*   **`pheromones[]`** — Active pheromone signals that modify swarm behaviour (e.g., `SECURITY_AUDIT`, `COST_LIMIT`, `QUALITY_GATE`).
*   **`taskGraph`** — Directed acyclic graph of decomposed tasks with dependency edges and parallel execution groups.
*   **`formicContext`** — Injected by the Coordinator. Contains relevant memories, Gene Pool patterns, and constitutional constraints.

### 4.4 Pheromone Coordination

Formic uses virtual pheromone signals for emergent swarm behaviour. Pheromones are broadcast events that any agent can emit or respond to:

| Pheromone | Emitter | Effect |
| :--- | :--- | :--- |
| `SECURITY_AUDIT` | SECOPS, DEVSECOPS | All agents pause; security review takes priority |
| `COST_LIMIT` | FINOPS | Agents switch to cheaper models; non-essential tasks deferred |
| `QUALITY_GATE` | QA_ENGINEER, AUDITOR | Pipeline blocks until quality issues resolved |
| `ESCALATION` | Any agent | Triggers Coordinator intervention for unresolvable issues |
| `STIGMERGY_UPDATE` | DISCOVERY | Notifies agents that codebase state has changed |

## 5. Tribunal Integration

Formic defines **5 conditions** that trigger a Tribunal review. As the meta-swarm, Formic Tribunals adjudicate both internal disputes and escalations from product swarms.

### 5.1 Trigger Conditions

| # | Condition | Trigger Rule | Jury Composition |
| :--- | :--- | :--- | :--- |
| 1 | **Budget Overrun** | Compute cost exceeds `constraints.budgetCeiling` by > 10% | `FINOPS` + `PROJECT_MANAGER` + `CHIEF_OF_STAFF` |
| 2 | **NFR Violation** | Agent output violates a constitutional constraint | `AUDITOR` + `ARCHITECT` + offending agent |
| 3 | **Security Finding** | SECOPS detects vulnerability, secret exposure, or dependency CVE | `SECOPS` + `DEVSECOPS` + `AUDITOR` |
| 4 | **Quality Gate Failure** | QA_ENGINEER reports test coverage < 80% or critical test failures | `QA_ENGINEER` + `BUILDER` + `ARCHITECT` |
| 5 | **Agent Conflict** | Two or more agents produce contradictory outputs for the same task | All conflicting agents + `CHIEF_OF_STAFF` + `AUDITOR` |

### 5.2 Tribunal Mechanics

*   **Voting:** Personality-weighted blind voting with sycophancy detection (see `../consensus/sycophancy.spec.md`).
*   **Personality Weighting:** Each agent's vote weight is derived from their AAPS personality profile (see Section 6). Agents with higher Conscientiousness and lower Agreeableness receive higher effective weight, reducing groupthink susceptibility.
*   **Constitutional Override:** If a Tribunal verdict conflicts with a constitutional constraint, the constraint takes precedence. The Tribunal is informed but cannot override NFRs.
*   **Escalation to Human:** If the Tribunal cannot reach consensus after two rounds, the mission is paused and routed to a human operator with full reasoning traces.
*   **Cross-Swarm Escalation:** Product swarm Tribunals that deadlock may escalate to Formic's CHIEF_OF_STAFF for meta-level adjudication.

## 6. Personality System (AAPS)

Each Formic agent has an **OCEAN personality profile** managed by the Agent Adaptive Personality System (AAPS). Refer to `../consensus/personality.spec.md` for the full specification.

Temperature is derived using: `temp = clamp(O * 0.6 + E * 0.2 - C * 0.15 - N * 0.1, 0.0, 1.0)`

### 6.1 Default Archetypes

| Agent | Archetype | O | C | E | A | N | Derived Temp |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Visionary | Gyne | 0.85 | 0.60 | 0.70 | 0.55 | 0.30 | ~0.53 |
| Project Manager | Pheromone | 0.50 | 0.80 | 0.75 | 0.70 | 0.25 | ~0.30 |
| Business Analyst | Forager | 0.65 | 0.70 | 0.60 | 0.65 | 0.35 | ~0.37 |
| Architect | Mason | 0.70 | 0.85 | 0.45 | 0.40 | 0.40 | ~0.34 |
| Builder | Ergate | 0.35 | 0.90 | 0.40 | 0.50 | 0.30 | ~0.13 |
| QA Engineer | Inspector | 0.30 | 0.90 | 0.35 | 0.25 | 0.65 | ~0.05 |
| Auditor | Sentinel | 0.25 | 0.95 | 0.30 | 0.20 | 0.70 | ~0.00 |
| DevOps | Carrier | 0.30 | 0.85 | 0.40 | 0.55 | 0.30 | ~0.10 |
| Service Manager | Trophallaxis | 0.40 | 0.80 | 0.60 | 0.65 | 0.35 | ~0.20 |
| FinOps | Replete | 0.25 | 0.90 | 0.30 | 0.35 | 0.50 | ~0.02 |
| SecOps | Guardian | 0.20 | 0.90 | 0.25 | 0.20 | 0.80 | ~0.00 |
| DevSecOps | Cleaner | 0.25 | 0.85 | 0.30 | 0.25 | 0.70 | ~0.01 |
| HR Manager | Nurse | 0.55 | 0.65 | 0.80 | 0.80 | 0.25 | ~0.37 |
| Designer | Morph | 0.90 | 0.50 | 0.75 | 0.60 | 0.20 | ~0.60 |
| Debugger | Soldier | 0.45 | 0.85 | 0.35 | 0.30 | 0.55 | ~0.16 |
| Researcher | Scout | 0.85 | 0.55 | 0.70 | 0.55 | 0.25 | ~0.54 |
| Discovery | Antenna | 0.80 | 0.60 | 0.55 | 0.50 | 0.30 | ~0.47 |
| Chief of Staff | — | 0.60 | 0.80 | 0.70 | 0.60 | 0.30 | ~0.35 |

**Design Notes:**
*   **Precision roles** (Builder, DevOps, Auditor, FinOps): High C (0.85-0.95), Low O — deterministic, repeatable outputs.
*   **Creative roles** (Designer, Researcher, Visionary): High O (0.85-0.90), High E — exploratory, high-temperature generation.
*   **Review roles** (QA Engineer, Auditor, SecOps): High N (0.55-0.80), High C — cautious, sceptical, hard to satisfy.
*   **Communication roles** (Project Manager, HR Manager, Service Manager): High E (0.60-0.80), High A (0.65-0.80) — collaborative, socially aware.
*   **Security roles** (SecOps, DevSecOps): High N (0.70-0.80), High C (0.85-0.90), Low A (0.20-0.25) — paranoid, uncompromising, contrarian.

### 6.2 Personality Evolution

Personality profiles evolve based on three feedback signals:

1.  **Mission Outcomes** — Agents on missions that succeed have their profiles reinforced. Agents on failed missions receive targeted adjustments based on root cause analysis (e.g., a Builder whose code caused a production incident has Conscientiousness increased).
2.  **Tribunal Verdicts** — Agents whose votes align with the final Tribunal verdict receive a small Conscientiousness boost. Agents on the losing side receive no penalty (to avoid risk-averse drift).
3.  **Cross-Swarm Feedback** — Product swarms that report poor coordination outcomes trigger adjustments to the Coordinator's Agreeableness and Extraversion (more assertive scheduling) or the Architect's Openness (more conservative designs).

## 7. Constitutional Governance

Formic enforces a set of **constitutional constraints** — immutable NFRs that no agent, Tribunal, or mission can override.

### 7.1 Core Constraints

| Constraint | Enforcement | Consequence of Violation |
| :--- | :--- | :--- |
| **No secret exposure** | All outputs scanned by SECOPS before delivery | Immediate mission halt + Tribunal |
| **Budget ceiling** | FINOPS monitors cumulative cost per mission | Pheromone `COST_LIMIT` + Tribunal if exceeded |
| **Test coverage floor** | QA_ENGINEER validates coverage before merge | Pipeline blocked until coverage restored |
| **Audit trail** | All agent actions logged with reasoning traces | Non-negotiable; agents cannot disable logging |
| **Human escalation** | Deadlocked Tribunals auto-escalate after 2 rounds | Mission paused; human operator notified |

### 7.2 Stigmergy

Agents practice **stigmergy** — indirect coordination through environment modification. Rather than explicit messaging, agents observe the codebase state:

*   A Builder that sees failing tests (left by QA_ENGINEER) will prioritise fixing them.
*   A SECOPS agent that sees a new dependency (added by Builder) will automatically scan it.
*   A DISCOVERY agent that sees undocumented code will generate documentation without being asked.

This emergent behaviour reduces coordination overhead and allows the swarm to self-organise around the most pressing work.

## 8. Cross-Swarm Interactions

As the meta-swarm, Formic is the nexus of all cross-swarm communication. All interactions flow through the Gene Pool (see `../core/gene_pool.spec.md`).

| Pattern Flow | Source | Destination | Example |
| :--- | :--- | :--- | :--- |
| Governance patterns | Formic | **All swarms** | Constitutional constraints, NFR profiles, audit templates |
| Quality standards | Formic | **All swarms** | Test coverage thresholds, code review checklists, security baselines |
| Code patterns | Formic | **All swarms** | Reusable architectural patterns, API design conventions |
| Pipeline templates | Formic | **All swarms** | CI/CD configurations, deployment recipes, monitoring dashboards |
| Domain patterns | **Provenance** | Formic | Luxury grading heuristics, authentication markers |
| Engagement patterns | **TalentOS** | Formic | Creator persona effectiveness, subscriber behaviour models |
| Safety policies | **SafetyOS** | Formic | Content moderation rules, compliance checklists |
| Market signals | **DVS** | Formic | Financial data quality patterns, valuation model feedback |

**Propagation Mechanism:** All cross-swarm interactions flow through the Gene Pool's async push/pull protocol. No direct swarm-to-swarm communication occurs — the Gene Pool acts as the sole intermediary, ensuring workspace isolation and audit traceability. Formic is unique in that it both produces governance patterns (outbound) and consumes domain patterns from every product swarm (inbound), making it the central nervous system of the colony.
