# LACP Layer 1: Transport & Routing (Detailed)

## 1. The Protocol Stack
LACP operates at **Layer 7 (Application)** but mimics lower-level transport protocols.

### 1.1 Wire Format
All messages are JSON-serialized `ContextObjects` transmitted via HTTP/2 (gRPC compatible).

```json
{
  "headers": {
    "x-lacp-ver": "1.0",
    "x-trace-id": "550e8400-e29b-41d4-a716-446655440000",
    "x-sender-id": "agent-social-bot-01",
    "x-ttl": 5
  },
  "body": { ... }
}
```

## 2. Rate Limiting (The Circuit Breaker)
*   **Goal:** Prevent recursive loops and cost spikes.
*   **Mechanism:** Token Bucket.
*   **Default:** 60 RPM (Requests Per Minute).
*   **Headers Returned:**
    *   `X-RateLimit-Limit: 60`
    *   `X-RateLimit-Remaining: 59`
    *   `X-RateLimit-Reset: 1678900000`

## 3. Error Handling
LACP defines standard error codes for agent failures:

| Code | Status | Meaning | Action |
| :--- | :--- | :--- | :--- |
| **4001** | `INVALID_CONTEXT` | Schema validation failed. | Reject & Alert Dev. |
| **4029** | `RATE_LIMIT` | Agent talking too much. | Backoff (Exponential). |
| **4031** | `SAFETY_VIOLATION` | Injection/PII detected. | **Quarantine Agent.** |
| **4041** | `SYCOPHANCY_DETECTED` | Vote discarded. | Penalize Trust Score. |
| **5000** | `HALLUCINATION` | Fact-check failed. | Retry with Temperature 0. |

## 4. The Runner (The "Switch")
The Runner (e.g., Formic) is the infrastructure executing the protocol.
*   **Dynamic Routing:** Can route `SocialBot` -> `SafetyOS` -> `Twitter` based on policy.
*   **Dead Letter Queue:** Failed messages are saved for human forensic analysis.