# LACP Specification (Modularized)

**Date:** January 29, 2026
**Version:** 1.1.0 (Refactored)

## 1. Introduction

The **LLM-Agent Communication Protocol (LACP)** is a strict Layer 7 protocol for autonomous agent swarms. This repository contains the modular specifications for the system.

## 2. Module Index

### 2.1 Core Kernel
*   **[Transport Layer](core/transport.spec.md):** Routing, Rate Limiting, and Headers.
*   **[Context & State](core/context.spec.md):** The `Context Object` and Immutability rules.

### 2.2 Security
*   **[PII Redaction](security/pii_redaction.spec.md):** NER-based data hygiene.
*   **[Injection Guard](security/injection_guard.spec.md):** Anti-prompt-injection heuristics.

### 2.3 Consensus (The Tribunal)
*   **[Tribunal Protocol](consensus/tribunal.spec.md):** The Commit-Reveal voting mechanism.
*   **[Sycophancy Detection](consensus/sycophancy.spec.md):** Vector-based groupthink mitigation.

### 2.4 Talents (Agent Roles)
*   **[Base Interface](talents/interface.spec.md):** Contract for all agents.
*   **[Architect](talents/architect.spec.md)**
*   **[Reviewer](talents/reviewer.spec.md)**
*   **[Security](talents/security.spec.md)**
*   **[Tester](talents/tester.spec.md)**

## 3. Schemas
Formal JSON schemas for data interchange:
*   [Context Object](schemas/context_object.json)
*   [Vote Object](schemas/vote_object.json)