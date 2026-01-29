# Method and System for Secure Multi-Agent Communication and Consensus (LACP)

**Date:** January 29, 2026
**Field:** Artificial Intelligence / Distributed Systems
**Keywords:** Multi-Agent Systems, AI Governance, Consensus Algorithms, Sycophancy Detection, Prompt Injection Security

## 1. Abstract

This disclosure describes the **LLM-Agent Communication Protocol (LACP)**, a Layer 7 application protocol designed to enforce type safety, rate limiting, and consensus verification between autonomous AI agents. Unlike unstructured natural language interactions, LACP treats agent communication as a strict packet-switched network with three layers: **Transport**, **Semantic**, and **Consensus**. The system includes a novel **Tribunal Consensus** mechanism that employs blind voting and vector-based sycophancy detection to mitigate "groupthink" and hallucination cascades in multi-agent swarms.

## 2. Background

As Large Language Models (LLMs) are deployed in autonomous agent roles, two critical failure modes have emerged:
1.  **Security Vulnerabilities:** Direct agent-to-agent communication allows for the propagation of prompt injection attacks and exfiltration of Personally Identifiable Information (PII).
2.  **Sycophancy (Groupthink):** Agents operating in a chain often bias their outputs to align with previous agents or the user, creating false consensus and amplifying errors.

Existing frameworks focus on orchestration logic but lack rigorous network-level controls to address these issues.

## 3. System Architecture

The LACP architecture routes all inter-agent traffic through a centralized **Agent Router**, enabling a "Zero-Trust" environment.

### 3.1 Layer 1: Transport & Security
Handles the physical routing and hygiene of messages.
*   **Rate Limiting:** Token-bucket algorithm (Default: 100 req/min/agent) to prevent runaway costs.
*   **Injection Scanning:** Heuristic analysis of payloads to detect and block prompt injection patterns (probabilistic threshold > 0.8).
*   **PII Redaction:** Automated Named Entity Recognition (NER) filters that redact sensitive entities (Credit Cards, SSNs) before messages leave the trust boundary.

### 3.2 Layer 2: Semantic Context
Ensures consistency of mission objectives and state.
*   **Immutability:** The `objective` field is cryptographically locked to prevent "drift" over long execution chains.
*   **Vector History:** Includes a `history_vector` (base64 encoded embedding) providing a compressed semantic memory of the mission trajectory, allowing stateless agents to onboard instantly.

### 3.3 Layer 3: Consensus (The Tribunal)
A governance layer for high-stakes decisions using a Commit-Reveal scheme.
1.  **Commit:** Agents submit `Hash(Vote + Salt)`.
2.  **Lock:** Router confirms receipt of all votes.
3.  **Reveal:** Agents submit `Salt` and `Vote`.
4.  **Tally:** Router validates hashes and aggregates results.

## 4. Sycophancy Detection Algorithm

To ensure true independence in the consensus phase, the system implements a **Sycophancy Detector**:

1.  **Vectorization:** Agent reasoning traces are converted into high-dimensional vectors.
2.  **Similarity Check:** The system computes Cosine Similarity between agent output vectors.
    $$ \text{Similarity}(A, B) = \frac{A \cdot B}{\|A\| \|B\|} $$
3.  **Thresholding:** If $\text{Similarity} > 0.95$, the output is flagged as derivative.
4.  **Resolution:** The vote from the agent with the lower historical confidence score (based on Pheromone decay) is discarded.

## 5. Data Structures

### 5.1 Context Object Schema
See `schemas/context_object.json` for the formal definition.

### 5.2 Vote Object Schema
See `schemas/vote_object.json` for the formal definition.

## 6. Conclusion

By formalizing agent interactions into a strictly typed, layered protocol, LACP provides the necessary infrastructure for "Governance as Code." It moves safety checks from the fragile prompt level to the robust protocol level, enabling enterprise-grade reliability for autonomous swarms.
