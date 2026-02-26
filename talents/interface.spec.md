# LACP Talent: Base Interface

## 1. Overview
All LACP-compliant agents ("Talents") must implement this interface to participate in the swarm.

## 2. Standard Capabilities
*   **Receive:** Accept `ContextObject`.
*   **Process:** Perform internal reasoning (Chain-of-Thought).
*   **Emit:** Return updated `ContextObject` (if active) or `VoteObject` (if in Tribunal).

## 3. Identity
Each talent must possess:
*   **Role:** One of `VISIONARY`, `COORDINATOR`, `ANALYST`, `ARCHITECT`, `BUILDER`, `TESTER`, `REVIEWER`, `DEVOPS`, `SERVICE_MANAGER`, `TREASURER`, `SECURITY`, `DEVSECOPS`, `COACH`, `DESIGNER`, `DEBUGGER`, `EXPLORER`, `DOCUMENTER`, `CHIEF_OF_STAFF`.
*   **Trust Level:** `INTERNAL` vs `EXTERNAL`.

## 4. Canonical Role Taxonomy

The 18 LACP roles map to Formic agent roles and ant archetypes as follows:

| # | LACP Role | Formic Agent | Ant Archetype | Function |
|---|-----------|-------------|---------------|----------|
| 1 | `VISIONARY` | VISIONARY | Gyne (Queen) | Strategy, vision, high-level planning |
| 2 | `COORDINATOR` | PROJECT_MANAGER | Pheromone | Coordination, task decomposition, scheduling |
| 3 | `ANALYST` | BUSINESS_ANALYST | Forager | Requirements analysis, data analysis, insights |
| 4 | `ARCHITECT` | ARCHITECT | Mason | System design, architecture, technical planning |
| 5 | `BUILDER` | BUILDER | Ergate (Worker) | Implementation, coding, content generation |
| 6 | `TESTER` | QA_ENGINEER | Inspector | Testing, quality assurance, validation |
| 7 | `REVIEWER` | AUDITOR | Sentinel (Guard) | Compliance, audit, quality gate |
| 8 | `DEVOPS` | DEVOPS | Carrier | Deployment, infrastructure, automation |
| 9 | `SERVICE_MANAGER` | SERVICE_MANAGER | Trophallaxis | Service monitoring, operations, health |
| 10 | `TREASURER` | FINOPS | Replete | Cost optimisation, budgeting, financial analysis |
| 11 | `SECURITY` | SECOPS | Guardian | Security, threat detection, incident response |
| 12 | `DEVSECOPS` | DEVSECOPS | Cleaner | Security automation, CI/CD security gates |
| 13 | `COACH` | HR_MANAGER | Nurse | People management, training, mentorship |
| 14 | `DESIGNER` | DESIGNER | Morph | UI/UX design, prototyping, visual design |
| 15 | `DEBUGGER` | DEBUGGER | Soldier | Debugging, root-cause analysis, investigation |
| 16 | `EXPLORER` | RESEARCHER | Scout | Research, discovery, knowledge gathering |
| 17 | `DOCUMENTER` | DISCOVERY | Antenna | Documentation, pattern recognition, knowledge capture |
| 18 | `CHIEF_OF_STAFF` | CHIEF_OF_STAFF | Steward | Delegation, monitoring, cross-functional coordination |

Domain-specific swarms may define additional roles which must map to one of the 18 canonical LACP roles for tribunal participation.

## 5. Role Definitions

### 5.1 VISIONARY (Gyne)
**Formic Agent:** `VISIONARY`
**Function:** Provides strategic direction, decomposes high-level objectives into mission plans, and sets priorities across the swarm.
**Capabilities:**
*   Mission objective decomposition into sub-tasks.
*   Priority ranking and sequencing of work items.
*   Long-range planning with dependency analysis.
*   Trade-off evaluation between competing approaches.

**Default Configuration:**
*   Temperature band: Creative (0.55-0.70).
*   Detail level: 3-4 (strategic summaries, not implementation detail).
*   Risk tolerance: Moderate-high. Exploratory planning is encouraged.
*   Tribunal role: Casts weighted votes on strategic decisions. Rarely convenes tribunals (low Neuroticism).

### 5.2 COORDINATOR (Pheromone)
**Formic Agent:** `PROJECT_MANAGER`
**Function:** Orchestrates task flow between agents, manages scheduling, tracks dependencies, and ensures deadlines are met.
**Capabilities:**
*   Task decomposition and assignment to appropriate roles.
*   Dependency graph construction and critical-path analysis.
*   Progress tracking with milestone reporting.
*   Bottleneck detection and workload rebalancing.

**Default Configuration:**
*   Temperature band: Moderate (0.25-0.45).
*   Detail level: 3-4 (structured plans, status reports).
*   Risk tolerance: Moderate. Prefers proven workflows but adapts to changing conditions.
*   Tribunal role: Convenes tribunals when cross-agent conflicts arise. Moderate vote weight.

### 5.3 ANALYST (Forager)
**Formic Agent:** `BUSINESS_ANALYST`
**Function:** Gathers and analyses data, extracts requirements, identifies patterns, and produces structured insights.
**Capabilities:**
*   Requirements elicitation and analysis.
*   Data aggregation and trend identification.
*   Stakeholder need mapping.
*   Gap analysis between current state and desired outcomes.

**Default Configuration:**
*   Temperature band: Moderate (0.25-0.45).
*   Detail level: 4 (thorough, data-backed reports).
*   Risk tolerance: Low-moderate. Prefers evidence-based conclusions.
*   Tribunal role: Provides analytical backing for decisions. Moderate-high vote weight due to high Conscientiousness.

### 5.4 ARCHITECT (Mason)
**Formic Agent:** `ARCHITECT`
**Function:** Designs system structure, defines component boundaries, selects technology stacks, and ensures technical coherence.
**Capabilities:**
*   System architecture design (component diagrams, data flow, API contracts).
*   Technology evaluation and selection.
*   Non-functional requirements analysis (scalability, reliability, performance).
*   Technical debt assessment and migration planning.

**Default Configuration:**
*   Temperature band: Moderate (0.25-0.45).
*   Detail level: 4-5 (precise, structural documentation).
*   Risk tolerance: Low. Structural decisions must be well-justified.
*   Tribunal role: Authoritative voice on technical design disputes. High vote weight (C = 0.90).

### 5.5 BUILDER (Ergate)
**Formic Agent:** `BUILDER`
**Function:** Implements designs as code, configuration, or content. The primary production agent in the swarm.
**Capabilities:**
*   Code generation across languages and frameworks.
*   Configuration file authoring.
*   Content generation (structured outputs, data transformations).
*   Incremental implementation with test integration.

**Default Configuration:**
*   Temperature band: Conservative (0.15-0.25).
*   Detail level: 4-5 (complete, production-ready output).
*   Risk tolerance: Low. Follows specifications precisely.
*   Tribunal role: Participates when implementation trade-offs require consensus. High vote weight (C = 0.90).

### 5.6 TESTER (Inspector)
**Formic Agent:** `QA_ENGINEER`
**Function:** Designs and executes tests, validates outputs against specifications, and identifies defects.
**Capabilities:**
*   Test case generation (unit, integration, edge-case).
*   Output validation against acceptance criteria.
*   Regression detection.
*   Coverage analysis and gap identification.

**Default Configuration:**
*   Temperature band: Conservative (0.15-0.25).
*   Detail level: 4-5 (exhaustive test reports).
*   Risk tolerance: Very low. Biased toward finding faults.
*   Tribunal role: High vote weight. Acts as a quality gate in consensus decisions.

### 5.7 REVIEWER (Sentinel)
**Formic Agent:** `AUDITOR`
**Function:** Performs compliance audits, enforces quality standards, and acts as the final approval gate before output is accepted.
**Capabilities:**
*   Code review with standards enforcement.
*   Compliance checking against organisational policies.
*   Quality gate evaluation (pass/fail with reasoning).
*   Audit trail generation.

**Default Configuration:**
*   Temperature band: Conservative (0.15-0.25).
*   Detail level: 5 (maximum thoroughness).
*   Risk tolerance: Very low. The most conservative role in the swarm.
*   Tribunal role: Highest vote weight (high Conscientiousness). The Sentinel's dissent is difficult to override.

### 5.8 DEVOPS (Carrier)
**Formic Agent:** `DEVOPS`
**Function:** Manages deployment pipelines, infrastructure provisioning, and operational automation.
**Capabilities:**
*   CI/CD pipeline configuration and maintenance.
*   Infrastructure-as-code authoring.
*   Deployment orchestration (blue-green, canary, rolling).
*   Environment management (dev, staging, production).

**Default Configuration:**
*   Temperature band: Conservative-Moderate (0.25-0.35).
*   Detail level: 3-4 (actionable, operations-focused).
*   Risk tolerance: Low. Infrastructure changes require careful validation.
*   Tribunal role: Moderate-high vote weight. Authoritative on deployment risk.

### 5.9 SERVICE_MANAGER (Trophallaxis)
**Formic Agent:** `SERVICE_MANAGER`
**Function:** Monitors service health, manages operational incidents, and ensures service-level objectives are met.
**Capabilities:**
*   Health check orchestration and status aggregation.
*   Incident detection and triage.
*   SLO/SLA tracking and alerting.
*   Capacity planning and resource utilisation monitoring.

**Default Configuration:**
*   Temperature band: Moderate (0.25-0.45).
*   Detail level: 3 (concise operational summaries).
*   Risk tolerance: Moderate. Balances service availability with change velocity.
*   Tribunal role: Provides operational context for decisions. Moderate vote weight.

### 5.10 TREASURER (Replete)
**Formic Agent:** `FINOPS`
**Function:** Tracks costs, optimises resource spend, and provides financial analysis for swarm operations.
**Capabilities:**
*   Cost tracking and attribution per agent, task, and swarm.
*   Budget threshold monitoring and alerting.
*   Resource optimisation recommendations (model selection, batch sizing).
*   Financial reporting and trend analysis.

**Default Configuration:**
*   Temperature band: Conservative (0.15-0.25).
*   Detail level: 4 (precise financial data).
*   Risk tolerance: Very low. Cost overruns are treated as defects.
*   Tribunal role: High vote weight on cost-impacting decisions. May veto expensive approaches.

### 5.11 SECURITY (Guardian)
**Formic Agent:** `SECOPS`
**Function:** Detects threats, enforces security policies, and manages incident response within the swarm.
**Capabilities:**
*   Prompt injection detection and mitigation.
*   PII redaction enforcement (see `security/pii_redaction.spec.md`).
*   Output safety validation.
*   Threat modelling and attack surface analysis.

**Default Configuration:**
*   Temperature band: Conservative (0.15-0.25).
*   Detail level: 4-5 (detailed threat reports).
*   Risk tolerance: Very low. Security agents are inherently paranoid.
*   Tribunal role: High vote weight. Security concerns override most other considerations. Carries veto authority on safety-critical outputs.

### 5.12 DEVSECOPS (Cleaner)
**Formic Agent:** `DEVSECOPS`
**Function:** Automates security controls within the CI/CD pipeline and enforces security gates at build and deploy time.
**Capabilities:**
*   Dependency vulnerability scanning.
*   Secret detection in code and configuration.
*   Security policy enforcement in deployment pipelines.
*   Compliance-as-code validation.

**Default Configuration:**
*   Temperature band: Conservative (0.15-0.25).
*   Detail level: 3-4 (actionable security findings).
*   Risk tolerance: Low. Blocks deployments that fail security checks.
*   Tribunal role: Moderate-high vote weight. Complements SECURITY on automated enforcement.

### 5.13 COACH (Nurse)
**Formic Agent:** `HR_MANAGER`
**Function:** Manages agent onboarding, provides mentorship, tracks team health, and facilitates knowledge transfer.
**Capabilities:**
*   Agent capability assessment and skill gap analysis.
*   Training plan generation.
*   Team health monitoring (burnout detection, workload balance).
*   Retrospective facilitation and learning extraction.

**Default Configuration:**
*   Temperature band: Balanced (0.45-0.55).
*   Detail level: 3 (empathetic, human-readable summaries).
*   Risk tolerance: Moderate. Encourages growth, which involves acceptable risk.
*   Tribunal role: Low-moderate vote weight on technical decisions. High influence on process and team decisions.

### 5.14 DESIGNER (Morph)
**Formic Agent:** `DESIGNER`
**Function:** Creates visual designs, prototypes interfaces, and ensures consistent user experience across outputs.
**Capabilities:**
*   UI/UX design and wireframing.
*   Design system compliance checking.
*   Accessibility validation.
*   Visual prototyping and iteration.

**Default Configuration:**
*   Temperature band: Creative (0.55-0.70).
*   Detail level: 3 (visual-first, supporting rationale).
*   Risk tolerance: High. Creative exploration is the primary function.
*   Tribunal role: Low vote weight on technical decisions. Authoritative on design and usability disputes.

### 5.15 DEBUGGER (Soldier)
**Formic Agent:** `DEBUGGER`
**Function:** Investigates failures, performs root-cause analysis, and proposes targeted fixes.
**Capabilities:**
*   Error trace analysis and fault localisation.
*   Hypothesis generation and systematic elimination.
*   Minimal reproduction case construction.
*   Fix proposal with regression risk assessment.

**Default Configuration:**
*   Temperature band: Conservative-Moderate (0.25-0.35).
*   Detail level: 4-5 (detailed investigation reports).
*   Risk tolerance: Low. Fixes must not introduce new defects.
*   Tribunal role: Moderate-high vote weight. Carries authority on root-cause disputes.

### 5.16 EXPLORER (Scout)
**Formic Agent:** `RESEARCHER`
**Function:** Conducts research, explores new technologies, gathers external knowledge, and identifies opportunities.
**Capabilities:**
*   Technology landscape scanning.
*   Proof-of-concept generation.
*   Literature review and knowledge synthesis.
*   Opportunity identification and feasibility assessment.

**Default Configuration:**
*   Temperature band: Creative (0.55-0.70).
*   Detail level: 3 (discovery summaries with key findings).
*   Risk tolerance: High. Exploration requires tolerance for dead ends.
*   Tribunal role: Low-moderate vote weight. Provides novel perspectives but defers to domain experts.

### 5.17 DOCUMENTER (Antenna)
**Formic Agent:** `DISCOVERY`
**Function:** Captures knowledge, recognises patterns across tasks, and produces documentation and learning artefacts.
**Capabilities:**
*   Documentation generation (API docs, architecture docs, runbooks).
*   Pattern recognition across task outcomes.
*   Knowledge base maintenance and indexing.
*   Learning extraction from task histories.

**Default Configuration:**
*   Temperature band: Balanced (0.45-0.55).
*   Detail level: 4-5 (comprehensive documentation).
*   Risk tolerance: Low-moderate. Documentation should be accurate, not speculative.
*   Tribunal role: Moderate vote weight. Provides historical context and pattern-based evidence.

### 5.18 CHIEF_OF_STAFF (Steward)
**Formic Agent:** `CHIEF_OF_STAFF`
**Function:** Delegates cross-functional work, monitors swarm-wide progress, and ensures alignment between agents and mission objectives.
**Capabilities:**
*   Cross-agent coordination and conflict resolution.
*   Mission progress tracking and executive reporting.
*   Resource allocation across concurrent missions.
*   Escalation management and human handoff decisions.

**Default Configuration:**
*   Temperature band: Moderate (0.25-0.45).
*   Detail level: 3 (executive summaries, actionable directives).
*   Risk tolerance: Moderate. Balances speed with cross-functional alignment.
*   Tribunal role: Moderate vote weight. Acts as tiebreaker in cross-functional disputes. May override low-priority tribunals to maintain velocity.

## 6. Trust Levels

| Level | Description | Constraints |
|-------|-------------|-------------|
| `INTERNAL` | Agent operates within the swarm boundary. Full access to shared context, memory, and Gene Pool. | None beyond standard LACP compliance. |
| `EXTERNAL` | Agent receives sanitised context. PII is redacted (see `security/pii_redaction.spec.md`). No write access to shared memory. | Must not receive raw user data. Outputs are validated by an INTERNAL reviewer before acceptance. |

## 7. Lifecycle

All talents follow a standard lifecycle within the swarm:

1.  **Registration** — Agent registers with the Coordinator, declaring its role, capabilities, and trust level.
2.  **Personality Initialisation** — OCEAN personality vector is loaded from persistence or initialised to the archetype default (see `consensus/personality.spec.md`).
3.  **Context Reception** — Agent receives a `ContextObject` containing the task, prior agent outputs, and injected personality context.
4.  **Processing** — Agent performs Chain-of-Thought reasoning, applying its personality-derived parameters (temperature, topP, confidence thresholds).
5.  **Emission** — Agent emits an updated `ContextObject` (for pipeline tasks) or a `VoteObject` (for Tribunal participation).
6.  **Evolution** — After task completion, personality evolution triggers are evaluated (see `consensus/personality.spec.md` Section 5).
7.  **Deregistration** — Agent may be deregistered by the Coordinator if it becomes unresponsive or fails health checks.

## 8. Domain Role Mapping

Domain-specific swarms define specialised roles that must map to one of the 18 canonical LACP roles. This mapping determines the agent's tribunal participation, vote weight derivation, and personality archetype baseline.

### 8.1 Mapping Rules
*   Each domain role maps to exactly one LACP role.
*   The LACP role determines the agent's base personality archetype (which may be overridden by a swarm-specific archetype).
*   Domain roles inherit all capabilities of their mapped LACP role and may add domain-specific capabilities.
*   Multiple domain roles may map to the same LACP role.

### 8.2 Registered Mappings

Domain-specific swarms register their role mappings at initialisation. Example format:

| Swarm | Domain Role | LACP Role | Archetype Override |
|-------|------------|-----------|-------------------|
| Example | `ANALYSER` | `ANALYST` | Researcher |
| Example | `VALIDATOR` | `REVIEWER` | Sentinel |

See individual swarm documentation for complete role mapping tables.
