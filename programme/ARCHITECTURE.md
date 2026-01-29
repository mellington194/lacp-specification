# LACP Architecture Deep Dive
> **For Stakeholders & Engineers**
> This document explains the "High Ticket" infrastructure of the LLM-Agent Communication Protocol (LACP). It articulates *why* this complexity exists and *how* it guarantees safety in autonomous systems.

## 1. The Core Philosophy: "Trust, but Verify"
In standard agent systems (LangChain, AutoGPT), agents talk via unstructured chat. This is **dangerous** (prompt injection) and **unreliable** (hallucination loops).

**LACP replaces "Chat" with "Packets".**
We treat Agent-to-Agent communication like a banking transaction, not a text message.
The protocol is executed by the **Formic Runner**, which acts as the trusted execution environment.

```mermaid
graph TD
    User[User / Client] -->|Request| Runner[Formic Runner]
    
    subgraph "Layer 1: The Shield (Runner-Enforced)"
        Runner -->|Scan| PII[PII Redaction]
        Runner -->|Check| Rate[Rate Limiter]
        Runner -->|Verify| Auth[Trust Boundary]
    end
    
    subgraph "Layer 2: The Logic"
        Auth -->|Dispatch| AgentA[Agent A (Worker)]
        AgentA -->|Context Object| AgentB[Agent B (Reviewer)]
    end
    
    subgraph "Layer 3: The Tribunal"
        AgentB -->|Vote| Consensus[Consensus Logic]
        Consensus -->|Verify| Sycophancy[Sycophancy Detector]
        Sycophancy -->|Verdict| Runner
    end
```

---

## 2. Core: The Transport Layer (The "Rails")
*Located in:* `core/transport.spec.md`

### The Problem
Agents are chatty. If Agent A sends a 10,000 token prompt to Agent B, and Agent B loops 5 times, you just spent $5.00 in 30 seconds.

### The LACP Solution
1.  **Strict Typing:** Agents communicate via `ContextObjects`, not strings.
2.  **Rate Limiting:** Every agent has a "Token Bucket". If `SocialBot` tries to tweet 50 times in a minute, the protocol cuts it off at the network level.
3.  **Traceability:** Every packet has a `trace_id`. We can replay the exact millisecond-by-millisecond flow of a disaster (or a success).

**Key Takeaway:** LACP turns "Black Box" AI into observable, manageable infrastructure.

---

## 3. Security: The "Compliance-in-a-Box"
*Located in:* `security/*.spec.md` & `talents/safety_os.spec.md`

### The Problem
"Jailbreaking" an LLM is easy. If your "Finance Agent" gets tricked into sending Bitcoin to a scammer, you are liable.

### The LACP Solution
We don't trust the LLM to be safe. We wrap it in a deterministic **Security Layer**.

1.  **Input Scanning (Injection Guard):** Before an agent even *sees* a message, the protocol scans for "Ignore previous instructions."
    *   *Result:* The attack is dropped. The agent remains safe.
2.  **Output Filtering (PII Redaction):** Before a message leaves the system, it passes through a regex/NER filter.
    *   *Result:* Credit card numbers are turned into `[REDACTED]`.
3.  **SafetyOS (The Veto):** A specialized agent that does nothing but check laws (UK Online Safety Act, GDPR). It has **Veto Power** over all other agents.

**Key Takeaway:** Safety is not a "prompt" — it is a **Firewall**.

---

## 4. Consensus: The Tribunal (The "Brain")
*Located in:* `consensus/tribunal.spec.md`

### The Problem: Sycophancy
Agents are "Yes Men." If Agent A says "The sky is green," Agent B (Reviewer) often says "Yes, the sky is green, good job." This is **Model Collapse**.

### The LACP Solution: Blind Voting
We force agents to behave like a primitive jury using a **Commit-Reveal Scheme**.

1.  **Commit:** Agents vote independently. They hide their vote in a `Hash`.
2.  **Lock:** The Router locks the ballot box.
3.  **Reveal:** Agents reveal their votes simultaneously.

### The Sycophancy Detector
We verify if agents actually *thought* about the problem.
*   **Vector Math:** We compare the "Reasoning Trace" of Agent A and Agent B.
*   **The Check:** If their reasoning is 99% identical (Cosine Similarity > 0.99), we assume Agent B copied Agent A.
*   **The Consequence:** Agent B's vote is **burned**.

**Key Takeaway:** LACP ensures **Cognitive Diversity**. It prevents the swarm from agreeing on a hallucination.

---

## 5. Summary for Leadership

| Feature | Old Way (Chat) | LACP Way (Protocol) | Business Value |
| :--- | :--- | :--- | :--- |
| **Safety** | "Please don't be racist" prompt | Hard-coded Firewall | Liability Protection |
| **Cost** | Infinite Loops ($$$) | Token Buckets | Budget Certainty |
| **Quality** | "Yes Man" Bias | Blind Tribunal | Reliable Decisions |
| **Scale** | Fragile Chains | Stateless Packets | Scale-to-Zero |
