# LACP Talent: SafetyOS (The Shield)

## 1. Overview
**SafetyOS** acts as the specialized **SECURITY** talent within the swarm. It is responsible for the "Compliance-in-a-Box" mandate, ensuring no content violates platform rules or legal statutes (e.g., UK Online Safety Act).

## 2. Unbundled Capabilities
Unlike the generic Security agent, SafetyOS provides API-first infrastructure:

### 2.1 Logic Routing (The Smart Router)
*   **Hash Match:** Instantly approves/blocks known content (Cost: $0.00).
*   **Local AI:** Runs on-device classifiers for Nudity/Gore (Cost: Low).
*   **Cloud Fallback:** Calls AWS/GCP only for ambiguous cases (Cost: High).

### 2.2 Compliance Engine
*   **Input:** `Context.payload` (Images, Text, Video Frames).
*   **Output:** `Verdict` (BLOCK / FLAGGED / SAFE) + `Legal_Citation`.
*   **Constraint:** Must be called *before* any External Egress (e.g., posting to Twitter).

## 3. LACP Interface
*   **Role:** `SECURITY`
*   **Tribunal Weight:** **Veto Power** (A `REJECT` vote from SafetyOS cannot be overruled by consensus).
*   **Payload Extension:**
    ```json
    "safety_report": {
      "verdict": "BLOCK",
      "reason": "CSAM_DETECTED",
      "confidence": 0.99,
      "routing_path": "local-hash-db"
    }
    ```
