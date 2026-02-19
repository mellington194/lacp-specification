# LACP Talent: TalentOS (The Creator Economy Swarm)

## 1. Overview

**TalentOS** is an AI-powered persona management platform for digital creators. It operates a 6-agent swarm that automates subscriber engagement, content creation, scheduling, and safety enforcement across multiple platforms — including OnlyFans, Fansly, Instagram, Telegram, and WhatsApp.

Unlike traditional chatbot frameworks that use a single model with prompt engineering, TalentOS decomposes the creator economy workflow into **specialised agents** that collaborate via LACP. Each agent handles a distinct concern — engagement, curation, moderation, media processing, scene generation, and scheduling — enabling creators to scale their presence without sacrificing authenticity or safety.

The core differentiator is the **Persona System**: a per-creator voice, style, and personality model that governs all outbound communication. Every agent references the creator's persona template, ensuring consistent tone and brand identity across platforms and interaction types.

## 2. Swarm Architecture

TalentOS operates a **content lifecycle pipeline** with a continuous safety overlay. The MODERATOR does not occupy a fixed stage — it monitors all stages concurrently and holds veto power at any point in the pipeline.

```
                    MODERATOR (sentinel)
                    │   Continuous safety monitoring
                    │   VETO power at any stage
                    │
                    ▼ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
                                                                │
Stage 1: CURATOR (explorer)                                     │
    │   Trend analysis, content selection,                      │
    │   engagement window identification                        │
    │                                                           │
    ├─────────────────────────────┐                             │
    ▼                             ▼                             │
Stage 2a: Content Path    Stage 2b: Engagement Path             │
    │                         │                                 │
    SCENE_GENERATOR           CHATTER (builder)                 │
    │   Multi-person scene    │   Subscriber DMs,               │
    │   generation (1-4       │   persona-consistent            │
    │   characters)           │   conversation                  │
    │                         │                                 │
    ▼                         │                                 │
    MEDIA_WORKER              │                                 │
    │   Watermark, format     │                                 │
    │   convert, safety scan  │                                 │
    │                         │                                 │
    └────────────┬────────────┘                                 │
                 ▼                                              │
Stage 3: SCHEDULER (coordinator)                                │
    │   Cross-platform timing, engagement                       │
    │   window optimisation, queue management                   │
    │                                                           │
    ▼ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
                 │
                 ▼
            PUBLISH (platform adapters)
```

**Design Rationale:**
*   Stages 2a (content creation) and 2b (subscriber engagement) execute in parallel — a creator's content pipeline and DM conversations are independent workflows that share a persona but not a dependency chain.
*   The MODERATOR operates as a **cross-cutting concern**, not a pipeline stage. It intercepts outputs from any agent before they reach platform adapters. This design ensures that a safety violation at any point — a SCENE_GENERATOR output, a CHATTER message, or a SCHEDULER queue — is caught before publication.
*   The MEDIA_WORKER sits between SCENE_GENERATOR and SCHEDULER because all visual content must be watermarked, format-converted, and safety-scanned before entering the publishing queue.
*   The pipeline supports multiple concurrent persona instances — a single TalentOS deployment can manage dozens of creator accounts simultaneously, each with isolated persona state.

## 3. Agent Roles

| Agent | LACP Role | Archetype | Model | Trust Level | Function |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Chatter** | `BUILDER` | Conversationalist | `claude-sonnet-4-5` | `INTERNAL` | Subscriber engagement via DMs, persona-consistent conversation, relationship management |
| **Curator** | `EXPLORER` | Trend Hunter | `gemini-2.0-flash` | `INTERNAL` | Content trend analysis, engagement pattern identification, content calendar planning |
| **Moderator** | `SECURITY` | Guardian | `gemini-2.5-pro` | `INTERNAL` | Safety policy enforcement, content compliance, subscriber screening, veto authority |
| **Media Worker** | `BUILDER` | Processor | `gemini-2.0-flash` | `INTERNAL` | Image/video watermarking, format conversion, resolution scaling, safety pre-scan |
| **Scene Generator** | `DESIGNER` | Artist | `flux-pro` / `sdxl` | `INTERNAL` | Multi-character scene generation (1-4 persons), location LoRA application, style transfer |
| **Scheduler** | `COORDINATOR` | Timekeeper | `gemini-2.0-flash` | `INTERNAL` | Cross-platform post timing, engagement window optimisation, queue prioritisation |

**Model Selection Notes:**
*   The **Chatter** uses `claude-sonnet-4-5` because subscriber engagement demands nuanced conversational ability, persona consistency, and emotional intelligence. Cheaper models produce noticeably less authentic interactions.
*   The **Moderator** uses `gemini-2.5-pro` for its superior reasoning on edge-case safety decisions — distinguishing between policy-compliant and policy-violating content requires contextual judgement, not just classification.
*   The **Scene Generator** uses specialised image generation models (`flux-pro`, `sdxl`) with creator-specific LoRA adapters rather than LLMs.
*   All other agents use `gemini-2.0-flash` for cost efficiency. Curation, media processing, and scheduling are structured tasks well-served by fast inference.

## 4. LACP Interface

### 4.1 Role Mapping

TalentOS domain-specific roles map to standard LACP roles:

| Domain Role | LACP Role | Justification |
| :--- | :--- | :--- |
| `CHATTER` | `BUILDER` | Builds conversation artefacts — messages, replies, engagement sequences |
| `CURATOR` | `EXPLORER` | Explores trends, discovers content opportunities, analyses engagement data |
| `MODERATOR` | `SECURITY` | Enforces safety policies — functionally equivalent to SafetyOS at the swarm level |
| `MEDIA_WORKER` | `BUILDER` | Builds processed media artefacts — watermarked, formatted, scanned outputs |
| `SCENE_GENERATOR` | `DESIGNER` | Designs visual content — multi-character scenes, styled imagery |
| `SCHEDULER` | `COORDINATOR` | Coordinates cross-platform publishing — timing, sequencing, queue management |

### 4.2 Trust Level

All agents operate at `INTERNAL` trust level. External platform interactions (posting to OnlyFans, sending Telegram messages) are handled by platform adapter services outside the agent boundary. Agents produce output artefacts; adapters handle egress.

### 4.3 Context Object

The mission payload conforms to the standard LACP Context Object (see `../core/context.spec.md`) with the following creator-economy-specific fields:

```json
{
  "mission_id": "talentos-{creator_id}-{action}-{timestamp}",
  "objective": "Engage subscribers / Generate content / Schedule posts",
  "payload": {
    "creatorId": "creator-uuid",
    "persona": {
      "voiceTemplate": "embedding-vector-ref",
      "styleGuide": "tone, vocabulary, emoji usage, taboo topics",
      "personalityProfile": { "O": 0.0, "C": 0.0, "E": 0.0, "A": 0.0, "N": 0.0 },
      "platformRules": {
        "onlyfans": { "contentPolicy": "...", "pricingTier": "premium" },
        "fansly": { "contentPolicy": "...", "pricingTier": "standard" },
        "instagram": { "contentPolicy": "...", "visibility": "public" }
      }
    },
    "subscriberContext": {
      "subscriberId": "sub-uuid",
      "conversationHistory": [],
      "engagementScore": 0.0,
      "tier": "free | standard | vip"
    },
    "formicContext": {
      "memories": [],
      "genePoolPatterns": [],
      "safetyPolicies": []
    }
  }
}
```

*   **`persona`** — The creator's complete personality model. All agents reference this to maintain voice consistency.
*   **`persona.voiceTemplate`** — Reference to a vector embedding of the creator's writing style, trained on their historical posts and messages.
*   **`persona.platformRules`** — Per-platform content policies. The MODERATOR enforces these; the SCHEDULER uses them for platform-specific formatting.
*   **`subscriberContext`** — For engagement missions. Contains conversation history and subscriber tier (which affects response priority and content access).
*   **`formicContext.safetyPolicies`** — Injected by the Coordinator from SafetyOS via the Gene Pool. Contains current content moderation rules.

### 4.4 Feature Flags

```
PERSONA_DRIFT_DETECTION=true
MULTI_PLATFORM_SYNC=true
SCENE_GENERATION_ENABLED=true
```

*   **`PERSONA_DRIFT_DETECTION`** — When enabled, CHATTER outputs are compared against the persona template using cosine similarity. Drift > 0.3 triggers a Tribunal.
*   **`MULTI_PLATFORM_SYNC`** — When enabled, SCHEDULER coordinates posting across all configured platforms. When disabled, each platform operates independently.
*   **`SCENE_GENERATION_ENABLED`** — When disabled, the content path skips SCENE_GENERATOR and only processes existing media through MEDIA_WORKER.

## 5. Tribunal Integration

TalentOS defines **5 domain-specific conditions** that trigger a Tribunal review. Given the platform's direct engagement with paying subscribers, Tribunal decisions prioritise safety and brand consistency over speed.

### 5.1 Trigger Conditions

| # | Condition | Trigger Rule | Jury Composition |
| :--- | :--- | :--- | :--- |
| 1 | **Safety Policy Violation** | Content flagged by MODERATOR or SafetyOS with confidence > 0.7 | `MODERATOR` + `CURATOR` + `SCHEDULER` |
| 2 | **Persona Drift** | CHATTER output diverges > 0.3 cosine similarity from persona template | `CHATTER` + `CURATOR` + `MODERATOR` |
| 3 | **Subscriber Complaint** | Engagement flagged as inappropriate or off-brand by subscriber report | `CHATTER` + `MODERATOR` + `SCHEDULER` |
| 4 | **Content Policy Conflict** | Platform-specific rules conflict with creator preferences (e.g., Instagram policy blocks content the creator approved) | `MODERATOR` + `CURATOR` + `SCHEDULER` |
| 5 | **Revenue Anomaly** | Subscriber churn exceeds 10% within a 24-hour window | `CURATOR` + `CHATTER` + `SCHEDULER` |

### 5.2 Tribunal Mechanics

*   **Voting:** Personality-weighted blind voting with sycophancy detection (see `../consensus/sycophancy.spec.md`).
*   **Moderator Veto:** The MODERATOR holds absolute veto power on safety-related verdicts (Triggers 1, 3). A `REJECT` vote from the MODERATOR on safety grounds cannot be overruled by consensus — mirroring SafetyOS's veto authority.
*   **Safety Default:** If the Tribunal cannot reach consensus on a content decision, the content is held (not published) rather than released. False negatives (blocking safe content) are preferred over false positives (releasing unsafe content).
*   **Escalation:** Items that fail Tribunal twice are routed to the creator for manual review, with all agent reasoning traces and the specific policy concern highlighted.
*   **Persona Recalibration:** Tribunal verdicts on persona drift (Trigger 2) feed back into the persona template, adjusting the voice embedding to reflect the creator's evolving style.

## 6. Personality System (AAPS)

Each TalentOS agent has an **OCEAN personality profile** managed by the Agent Adaptive Personality System (AAPS). Refer to `../consensus/personality.spec.md` for the full specification.

Temperature is derived using: `temp = clamp(O * 0.6 + E * 0.2 - C * 0.15 - N * 0.1, 0.0, 1.0)`

### 6.1 Default Archetypes

| Agent | Archetype | O | C | E | A | N | Derived Temp |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Chatter | Conversationalist | 0.65 | 0.55 | 0.85 | 0.75 | 0.30 | ~0.45 |
| Curator | Trend Hunter | 0.80 | 0.70 | 0.50 | 0.55 | 0.35 | ~0.44 |
| Moderator | Guardian | 0.25 | 0.90 | 0.30 | 0.25 | 0.75 | ~0.00 |
| Media Worker | Processor | 0.30 | 0.85 | 0.25 | 0.50 | 0.40 | ~0.06 |
| Scene Generator | Artist | 0.90 | 0.45 | 0.70 | 0.55 | 0.20 | ~0.59 |
| Scheduler | Timekeeper | 0.40 | 0.80 | 0.45 | 0.55 | 0.35 | ~0.18 |

**Design Notes:**
*   **Chatter** (High E: 0.85, High A: 0.75, moderate O: 0.65): Warm, engaging, and authentic. The highest Extraversion in the swarm reflects its role as the primary subscriber-facing agent. Moderate Openness allows creative responses without straying from the persona template.
*   **Curator** (High O: 0.80, High C: 0.70, moderate E: 0.50): Trend-aware and methodical. High Openness enables trend discovery; high Conscientiousness ensures recommendations are data-backed, not speculative.
*   **Moderator** (High C: 0.90, High N: 0.75, Low A: 0.25): Strict, vigilant, and contrarian. The lowest Agreeableness in the swarm is by design — a Moderator that agrees too easily is a liability. High Neuroticism makes it cautious and sensitive to edge cases. Derived temperature of ~0.00 ensures near-deterministic safety decisions.
*   **Media Worker** (High C: 0.85, Low O: 0.30, Low E: 0.25): Precise, efficient, and deterministic. Media processing requires consistent, repeatable outputs — watermarks must be pixel-perfect, format conversions must be lossless.
*   **Scene Generator** (High O: 0.90, High E: 0.70, Low N: 0.20): Creative, bold, and experimental. The highest Openness in the swarm enables diverse visual output. Low Neuroticism means it takes creative risks without hesitation.
*   **Scheduler** (High C: 0.80, moderate A: 0.55, Low N: 0.35): Reliable, balanced, and data-driven. Scheduling is fundamentally a constraint-satisfaction problem — high Conscientiousness ensures optimal timing, while moderate Agreeableness allows flexibility when platforms have conflicting requirements.

### 6.2 Personality Evolution

Personality profiles evolve based on three feedback signals:

1.  **Subscriber Engagement Metrics** — CHATTER profiles are adjusted based on message open rates, response rates, and subscriber retention. A Chatter whose conversations correlate with subscriber churn has Agreeableness reduced (more authentic, less people-pleasing) and Conscientiousness increased (more thoughtful responses).
2.  **Tribunal Outcomes** — Agents whose votes align with the final Tribunal verdict receive a small Conscientiousness boost. Agents on the losing side receive no penalty (to avoid risk-averse drift).
3.  **Creator Feedback** — Creators can directly approve or reject agent outputs. Approvals reinforce current profiles. Rejections trigger targeted adjustments: if a creator rejects a SCENE_GENERATOR output as "too conservative," Openness increases; if they reject a CHATTER message as "too casual," Conscientiousness increases.

## 7. Persona System

The Persona System is TalentOS's domain-specific intelligence layer — equivalent to Provenance's Vision Integration but for creator identity rather than product attributes.

### 7.1 Voice Template

*   **Representation:** Dense vector embedding trained on the creator's historical messages, posts, and approved content.
*   **Training:** Fine-tuned on minimum 500 messages per creator. Updated weekly with new approved content.
*   **Usage:** Injected into CHATTER's context as a style constraint. The model is instructed to match the voice template's tone, vocabulary, and communication patterns.
*   **Drift Detection:** Every CHATTER output is compared against the template. Cosine similarity < 0.7 triggers a Tribunal (see Section 5.1, Trigger 2).

### 7.2 Subscriber Memory

*   **Per-Subscriber Context:** Conversation history, engagement patterns, content preferences, and tier information maintained per subscriber.
*   **Memory Window:** Last 50 messages per subscriber, plus a compressed summary of the full history.
*   **Privacy:** Subscriber data is workspace-isolated. No cross-creator data leakage is possible — enforced by Formic's constitutional constraints.

### 7.3 Platform Adaptation

*   **Per-Platform Rules:** Each platform has distinct content policies, formatting requirements, and engagement norms. The MODERATOR enforces platform-specific rules; the SCHEDULER handles platform-specific formatting.
*   **Cross-Platform Consistency:** While formatting adapts per platform, the creator's voice remains consistent. The Persona System normalises tone across platforms — a message on Telegram and a caption on Instagram should sound like the same person.

## 8. Cross-Swarm Interactions

TalentOS participates in the broader Formic ecosystem via the Gene Pool (see `../core/gene_pool.spec.md`).

| Pattern Flow | Source | Destination | Example |
| :--- | :--- | :--- | :--- |
| Safety policies | **SafetyOS** | TalentOS | Content moderation rules, platform-specific compliance checklists, prohibited content classifiers |
| Visual guidelines | **Artifex** | TalentOS | Scene composition rules, colour palettes, lighting presets for SCENE_GENERATOR |
| Engagement patterns | TalentOS | **Gene Pool** | Persona effectiveness metrics, optimal posting times, subscriber churn predictors |
| Subscriber analytics | TalentOS | **DVS** | Subscriber behaviour patterns, revenue forecasting signals, churn correlation data |
| Grading patterns | **Provenance** | TalentOS | Luxury item knowledge for creators in the fashion/luxury space — styling context for content |
| Governance patterns | **Formic** | TalentOS | Constitutional constraints, audit templates, NFR profiles |

**Propagation Mechanism:** All cross-swarm interactions flow through the Gene Pool's async push/pull protocol. No direct swarm-to-swarm communication occurs — the Gene Pool acts as the sole intermediary, ensuring workspace isolation and audit traceability. TalentOS is a net producer of engagement intelligence and a net consumer of safety policies, making SafetyOS its most critical upstream dependency.
