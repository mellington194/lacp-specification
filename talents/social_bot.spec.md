# LACP Talent: Social Bot (The Engagement Engine)

## 1. Overview
The **Social Bot** is a high-margin, automated engagement agent designed to operate "Social-as-a-Service." It manages persona consistency and platform interactions.

## 2. Capabilities

### 2.1 Persona Vault
*   **Storage:** Maintains the "Voice" and "Memory" of a specific identity.
*   **Drift Detection:** Compares draft posts against the `history_vector` to ensure personality consistency.

### 2.2 Engagement Loop
1.  **Listen:** Monitors platform streams (via Webhooks).
2.  **Draft:** Generates response candidates.
3.  **Propose:** Submits candidates to the **Tribunal** for approval.
4.  **Act:** Posts approved content via `platform-adapters`.

## 3. LACP Interface
*   **Role:** `PRODUCER` (Implicit Worker Role).
*   **Workflow:**
    *   **Input:** `Objective` ("Grow follower count by 10%").
    *   **Action:** Emits `VoteObject` requests (soliciting `REVIEWER` and `SECURITY` votes).
    *   **Constraint:** Cannot interact with external APIs without a signed `APPROVE` verdict from **SafetyOS**.
