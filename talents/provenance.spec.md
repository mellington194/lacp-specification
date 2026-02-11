# LACP Talent: Provenance (Luxury Authentication Pipeline)

## 1. Overview

**Provenance** is a luxury retail listing platform powered by a 6-agent AI swarm. It performs automated condition grading, brand authentication, pricing, and listing copy generation for pre-owned luxury goods — including handbags, shoes, clothing, watches, jewellery, and accessories.

Unlike the original draft (which positioned Provenance as a single "Merchant" agent), the production architecture is a **self-contained swarm** that integrates with the broader Formic ecosystem via LACP. Each agent is a domain specialist, and the pipeline is orchestrated by the Formic Coordinator using standard LACP transport.

## 2. Swarm Architecture

Provenance operates a **5-stage sequential pipeline** with 6 specialised agents. Stages 2a and 2b execute in parallel to reduce latency.

```
Stage 1: IDENTIFIER (Scholar)
    │   Brand, model, material, colour, category detection
    │
    ├──────────────────────────────┐
    ▼                              ▼
Stage 2a: GRADER (Sentinel)    Stage 2b: AUTHENTICATOR (Paladin)
    │   TRR/VC condition grades    │   Serial codes, date codes,
    │   Defect bounding boxes      │   authenticity indicators
    │                              │
    └──────────────┬───────────────┘
                   ▼
Stage 3: APPRAISER (Merchant)
    │   Market pricing based on condition + authenticity
    │
    ▼
Stage 4: COPYWRITER (Rogue)
    │   Listing title, description, tags, highlights
    │
    ▼
Stage 5: QA_REVIEWER (Protector)
        Cross-validates all outputs, flags inconsistencies
```

**Design Rationale:**
*   Stages 2a/2b are independent — grading does not require authentication results and vice versa. Parallel execution reduces wall-clock time by ~40%.
*   Each stage receives the cumulative output of all prior stages, allowing downstream agents to reference upstream conclusions.
*   The pipeline is idempotent: re-running with the same images and context produces deterministic results (given fixed model versions).

## 3. Agent Roles

| Agent | LACP Role | Archetype | Model | Trust Level | Function |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Identifier** | `EXPLORER` | Scholar | `gemini-2.0-flash` | `INTERNAL` | Brand, model, material, colour, and category detection from product images |
| **Grader** | `BUILDER` | Sentinel | `gemini-2.0-flash` | `INTERNAL` | TRR/VC condition grades, defect detection with bounding boxes |
| **Authenticator** | `VALIDATOR` | Paladin | `gemini-2.5-pro` | `INTERNAL` | Serial codes, date codes, authenticity indicators, confidence scoring |
| **Appraiser** | `BUILDER` | Merchant | `gemini-2.0-flash` | `INTERNAL` | Market price estimation with price ranges, confidence, and pricing notes |
| **Copywriter** | `DOCUMENTER` | Rogue | `gemini-2.0-flash` | `INTERNAL` | SEO-optimised listing copy: title, description, tags, highlights |
| **QA Reviewer** | `REVIEWER` | Protector | `gemini-2.5-pro` | `INTERNAL` | Cross-validation, consistency checks, final confidence scoring |

**Model Selection Notes:**
*   The `Authenticator` and `QA Reviewer` use `gemini-2.5-pro` because their tasks require higher reasoning capability — authentication demands nuanced visual inspection, and QA must synthesise all prior outputs.
*   All other agents use `gemini-2.0-flash` for cost efficiency and speed. Flash models are sufficient for structured extraction and generation tasks.

## 4. LACP Interface

### 4.1 Role Mapping

Provenance domain-specific roles map to Formic generic LACP roles:

| Domain Role | LACP Role | Justification |
| :--- | :--- | :--- |
| `IDENTIFIER` | `EXPLORER` | Explores and discovers product attributes from raw images |
| `GRADER` | `BUILDER` | Builds the condition assessment — a structured output artefact |
| `AUTHENTICATOR` | `VALIDATOR` | Validates authenticity claims against known markers |
| `APPRAISER` | `BUILDER` | Builds the pricing estimate — another structured output |
| `COPYWRITER` | `DOCUMENTER` | Documents the product as a customer-facing listing |
| `QA_REVIEWER` | `REVIEWER` | Reviews and cross-validates all upstream outputs |

### 4.2 Trust Level

All agents operate at `INTERNAL` trust level. Provenance does not expose any agent to external egress — all outputs are returned to the calling application, which manages platform publishing separately.

### 4.3 Context Object

The mission payload conforms to the standard LACP Context Object (see `../core/context.spec.md`) with the following domain-specific fields:

```json
{
  "mission_id": "ls-grade-{item_id}-{timestamp}",
  "objective": "Grade, authenticate, price, and list luxury item",
  "payload": {
    "imageUrls": [
      "https://storage.example.com/items/123/front.jpg",
      "https://storage.example.com/items/123/back.jpg",
      "https://storage.example.com/items/123/serial.jpg"
    ],
    "category": "handbags",
    "brandHint": "Louis Vuitton",
    "priority": "standard",
    "formicContext": {
      "memories": [],
      "genePoolPatterns": [],
      "previousGrades": []
    }
  }
}
```

*   **`imageUrls[]`** — Array of product image URLs (minimum 1, recommended 4-8 for accuracy).
*   **`category`** — One of: `handbags`, `shoes`, `clothing`, `watches`, `jewellery`, `accessories`.
*   **`brandHint`** — Optional. Seller-provided brand name to prime the Identifier. May be overridden if visual evidence contradicts.
*   **`priority`** — `standard` (default) or `express`. Express items skip the Gene Pool enrichment step.
*   **`formicContext`** — Injected by the Coordinator. Contains relevant memories, Gene Pool patterns, and historical grades for the same brand/model.

### 4.4 Feature Flag

```
USE_SWARM_GRADING=true
```

*   **`true`** — Activates the full 6-agent swarm pipeline.
*   **`false`** — Falls back to a single Gemini call that performs all tasks in one prompt. Used for cost reduction during low-priority batch processing.
*   **Auto-fallback:** If the swarm pipeline fails (timeout, model error, or coordinator crash), the system automatically retries with the single-call fallback. This ensures zero-downtime grading even during swarm outages.

## 5. Tribunal Integration

Provenance defines **5 domain-specific conditions** that trigger a Tribunal review. When triggered, the pipeline pauses and a Tribunal is convened using the standard Commit-Reveal flow (see `../consensus/tribunal.spec.md`).

### 5.1 Trigger Conditions

| # | Condition | Trigger Rule | Jury Composition |
| :--- | :--- | :--- | :--- |
| 1 | **Low Confidence** | Any agent returns `confidence < 0.6` | `QA_REVIEWER` + originating agent + `APPRAISER` |
| 2 | **Grade/Auth Conflict** | Grader says "Excellent" but Authenticator flags concerns, or vice versa | `QA_REVIEWER` + `GRADER` + `AUTHENTICATOR` |
| 3 | **High-Value Item** | Appraiser estimates price > £5,000 | Full panel (all 6 agents) |
| 4 | **Rare/Unusual Item** | Identifier cannot match to a known model with confidence > 0.8 | `IDENTIFIER` + `AUTHENTICATOR` + `QA_REVIEWER` |
| 5 | **Ambiguous Defects** | Grader detects defects but cannot classify severity (e.g., patina vs stain on leather) | `GRADER` + `QA_REVIEWER` + `APPRAISER` |

### 5.2 Tribunal Mechanics

*   **Voting:** Personality-weighted blind voting with sycophancy detection (see `../consensus/sycophancy.spec.md`).
*   **Personality Weighting:** Each agent's vote weight is derived from their AAPS personality profile (see Section 6). Agents with higher Conscientiousness and lower Agreeableness receive higher effective weight, as they are less susceptible to groupthink.
*   **Safety Default:** If the Tribunal cannot reach consensus (e.g., tie vote), the item is flagged for human review rather than auto-published.
*   **Escalation:** Items that fail Tribunal twice are routed to a human expert queue with all agent reasoning traces attached.

## 6. Personality System (AAPS)

Each Provenance agent has an **OCEAN personality profile** managed by the Agent Adaptive Personality System (AAPS). Refer to `../consensus/personality.spec.md` for the full specification.

### 6.1 Default Archetypes

| Agent | Archetype | O | C | E | A | N | Derived Temp |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Identifier | Scholar | 0.8 | 0.7 | 0.4 | 0.6 | 0.3 | 0.45 |
| Grader | Sentinel | 0.3 | 0.9 | 0.3 | 0.2 | 0.4 | 0.20 |
| Authenticator | Paladin | 0.4 | 0.9 | 0.3 | 0.1 | 0.5 | 0.15 |
| Appraiser | Merchant | 0.6 | 0.7 | 0.6 | 0.5 | 0.4 | 0.40 |
| Copywriter | Rogue | 0.9 | 0.4 | 0.8 | 0.7 | 0.2 | 0.75 |
| QA Reviewer | Protector | 0.5 | 0.9 | 0.3 | 0.3 | 0.6 | 0.18 |

**Calibration Note:** Default archetypes are calibrated so that derived temperatures match the pre-AAPS hardcoded values used in Provenance v1. This ensures backward-compatible behaviour when the personality system is first enabled.

### 6.2 Personality Evolution

Personality profiles evolve based on three feedback signals:

1.  **Human Corrections** — When a human expert overrides an agent's output, the agent's profile is nudged toward the correction direction (e.g., a Grader that is too lenient has Agreeableness reduced).
2.  **Tribunal Outcomes** — Agents whose votes align with the final Tribunal verdict receive a small Conscientiousness boost. Agents on the losing side receive no penalty (to avoid risk aversion).
3.  **Accuracy Feedback** — Post-sale returns and customer complaints feed back into the system. If a Grader consistently over-grades items that get returned, their Conscientiousness increases and Openness decreases (making them more conservative).

## 7. Vision Integration

Provenance augments LLM-based agents with dedicated vision models for structured visual analysis.

### 7.1 FashionSigLIP

*   **Model:** `Marqo/marqo-fashionSigLIP`
*   **Purpose:** Visual embeddings for fashion items. Used by the Identifier for brand/model similarity search and by the Appraiser for comparable item retrieval.
*   **Deployment:** GPU-accelerated on RTX 3060. Inference time ~50ms per image.
*   **Integration:** Embeddings are stored alongside item records and injected into agent context via `formicContext.memories`.

### 7.2 Condition Classifier

*   **Architecture:** Fine-tuned vision model trained on Provenance's proprietary grading dataset.
*   **Classes:** 5-class condition prediction (`New`, `Excellent`, `Very Good`, `Good`, `Fair`).
*   **Usage:** Provides a structured condition signal to the Grader agent, which combines it with its own LLM-based analysis for the final grade.
*   **Training Data:** ~12,000 labelled images across all categories, with human-verified condition grades.

### 7.3 Attribute Classifier

*   **Architecture:** Multi-label fashion attribute detection model.
*   **Attributes:** Colour, material, hardware type, closure type, pattern, and ~40 other fashion-specific attributes.
*   **Usage:** Feeds structured attribute data to the Identifier, reducing hallucination risk for factual attributes.

## 8. Cross-Swarm Interactions

Provenance does not operate in isolation. Via the Gene Pool (see `../core/gene_pool.spec.md`), learnings propagate to and from other swarms in the ecosystem.

| Pattern Flow | Source | Destination | Example |
| :--- | :--- | :--- | :--- |
| Grading patterns | Provenance | **TalentOS** (via Gene Pool) | "Chanel caviar leather in good condition typically shows X" — helps styling content generation |
| Authentication markers | Provenance | **SafetyOS** | Counterfeit detection patterns feed into broader safety policies |
| Pricing trends | Provenance | **Deep Value Screener** | Luxury resale pricing data enriches market analysis |
| Visual guidelines | Provenance | **Artifex** | Product photography composition rules derived from high-performing listings |
| Styling context | TalentOS | **Provenance** | Fashion trend data informs Copywriter tone and keyword selection |
| Safety policies | SafetyOS | **Provenance** | Updated counterfeit blacklists injected into Authenticator context |

**Propagation Mechanism:** All cross-swarm interactions flow through the Gene Pool's async push/pull protocol. No direct swarm-to-swarm communication occurs — the Gene Pool acts as the sole intermediary, ensuring workspace isolation and audit traceability.
