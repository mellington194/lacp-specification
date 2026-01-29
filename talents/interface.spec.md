# LACP Talent: Base Interface

## 1. Overview
All LACP-compliant agents ("Talents") must implement this interface to participate in the swarm.

## 2. Standard Capabilities
*   **Receive:** Accept `ContextObject`.
*   **Process:** Perform internal reasoning (Chain-of-Thought).
*   **Emit:** Return updated `ContextObject` (if active) or `VoteObject` (if in Tribunal).

## 3. Identity
Each talent must possess:
*   **Role:** One of `REVIEWER`, `SECURITY`, `ARCHITECT`, `TESTER`.
*   **Trust Level:** `INTERNAL` vs `EXTERNAL`.
