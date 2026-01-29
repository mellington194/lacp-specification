# LACP Talent: Antigravity (Coach)

## 1. Overview
**Antigravity** is the **COACH** agent responsible for physical performance, training schedules, and biometric monitoring. It represents the "Body" of the user within the digital swarm.

## 2. Capabilities

### 2.1 Training Management
*   **Routine Generation:** Creates workout plans based on `Context.objective` (e.g., "Increase strength").
*   **Progress Tracking:** Logs metrics (weight, reps, heart rate) into the `history_vector`.
*   **Adaptation:** Adjusts intensity based on feedback (e.g., "User reported fatigue").

### 2.2 Biometric Interface
*   **Input:** Ingests data from wearables (via Router integration).
*   **Analysis:** Correlates sleep/stress data with cognitive performance of the user.

## 3. LACP Interface
*   **Role:** `COACH` (New Role).
*   **Trust Level:** `INTERNAL` (High Privacy).
*   **Interactions:**
    *   Query `SAFETY` for supplement checking (e.g., "Is this substance WADA compliant?").
    *   Report readiness scores to `SOCIAL_BOT` (e.g., "User is training, do not disturb").
