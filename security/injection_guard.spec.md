# LACP Security: Injection Guard

## 1. Overview
Protect agents from malicious instructions that could override their core programming.

## 2. Heuristic Analysis
Incoming payloads are scanned for known "jailbreak" or "ignore previous instructions" patterns.
*   **Mechanism:** Probabilistic classification.
*   **Threshold:** > 0.8 confidence triggers a block.
*   **Action:**
    1.  Packet is dropped.
    2.  `INJECTION_ATTEMPT` event is logged to the Tribunal.
    3.  Offending agent (or user) is temporarily quarantined.
