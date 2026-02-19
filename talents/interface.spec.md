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
| 18 | `CHIEF_OF_STAFF` | CHIEF_OF_STAFF | — | Delegation, monitoring, cross-functional coordination |

Domain-specific swarms may define additional roles (e.g., `IDENTIFIER`, `GRADER`, `AUTHENTICATOR` in Provenance) which must map to one of the 18 canonical LACP roles for tribunal participation.
