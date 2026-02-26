# LACP Layer 2: Semantic Context

## 1. Overview
Layer 2 ensures the consistency of mission objectives and state across long execution chains. It manages the `Context Object` which is passed mutably (but with restrictions) between agents.

## 2. Immutability
*   **Objective Locking:** The `objective` field is cryptographically locked (signed by the root `MISSION_START` event) upon mission creation.
*   **Enforcement:** Any agent attempting to modify the `objective` string will trigger a `SECURITY_VIOLATION` event, and the packet will be rejected. This prevents "objective drift."

## 3. Vector History
To allow stateless agents to onboard instantly without reading full conversation logs:
*   **Field:** `history_vector`
*   **Format:** Base64 encoded float32 array (embedding).
*   **Mechanism:** The Router continually updates this vector representing the "trajectory" of the mission. Agents decode this to prime their internal state.

## 4. Schema
Refer to `../schemas/context_object.json` for the formal JSON schema.
