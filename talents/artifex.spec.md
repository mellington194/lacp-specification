# LACP Talent: Artifex (Media Generation Pipeline)

## 1. Overview

**Artifex** is a unified AI media generation swarm that orchestrates image, video, and audio production through a 5-agent pipeline. It handles prompt engineering, multi-modal generation (Imagen 3, Veo 2, ElevenLabs), post-processing with provenance watermarking, and output validation — all driven by persona presets that encode style, tone, and visual parameters.

Unlike a monolithic generation endpoint, Artifex decomposes media production into discrete, auditable stages. A Director agent selects models and presets, a Scriptwriter engineers prompts, parallel generation agents produce media assets, an Editor applies watermarking and post-processing, and a QA agent validates the final output. This architecture supports 9 distinct persona presets, each with calibrated generation parameters.

## 2. Swarm Architecture

Artifex operates a **4-stage pipeline** with 5 specialised agents. Stage 3 executes generation tasks in parallel across modalities to reduce wall-clock time.

```
Stage 1: DIRECTOR (Architect)
    |   Orchestration, model selection, persona preset loading
    |
    v
Stage 2: SCRIPTWRITER (Documenter)
    |   Prompt engineering, scene descriptions, storyboards
    |
    |----------------------------+----------------------------+
    v                            v                            v
Stage 3a: NARRATOR (Builder)  Stage 3b: Image Gen        Stage 3c: Video Gen
    |   TTS via ElevenLabs       |   Imagen 3                |   Veo 2
    |   Voice cloning            |   ComfyUI workflows       |   Scene composition
    |   Ambient audio            |                            |
    +----------------------------+----------------------------+
                                 |
                                 v
Stage 4: EDITOR (Builder)
    |   Watermarking (LSB, DCT, DWT), format conversion, quality enhancement
    |
    v
Stage 5: QA (Reviewer)
        Content safety, watermark integrity, format compliance, persona consistency
```

**Design Rationale:**
*   Stage 3 branches are independent — TTS, image, and video generation have no data dependencies on each other. Parallel execution reduces total latency by ~60% compared to sequential generation.
*   The Editor applies three watermarking algorithms (LSB, DCT, DWT) to establish content provenance. All three survive common compression and re-encoding operations.
*   9 persona presets (not 6 as some earlier documentation claims) each encode style parameters, tone profiles, and visual generation constraints.

## 3. Agent Roles

| Agent | LACP Role | Archetype | Model | Trust Level | Function |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Director** | `ARCHITECT` | Commander | `claude-sonnet-4-5` | `INTERNAL` | Orchestrates the full media pipeline, selects models and presets, manages persona-driven generation |
| **Scriptwriter** | `DOCUMENTER` | Storyteller | `claude-sonnet-4-5` | `INTERNAL` | Generates narrative prompts, scene descriptions, and storyboards from high-level briefs |
| **Narrator** | `BUILDER` | Performer | `elevenlabs-v2` | `EXTERNAL` | Text-to-speech generation, voice cloning, ambient audio mixing via ElevenLabs API |
| **Editor** | `BUILDER` | Craftsman | `comfyui-local` | `INTERNAL` | Post-processing: watermarking (LSB, DCT, DWT), format conversion, quality enhancement |
| **QA** | `REVIEWER` | Inspector | `claude-sonnet-4-5` | `INTERNAL` | Output validation: content safety, watermark integrity, format compliance, persona consistency |

**Model Selection Notes:**
*   The Director, Scriptwriter, and QA agents use `claude-sonnet-4-5` for reasoning capability — orchestration, creative prompting, and cross-validation all require nuanced judgement.
*   The Narrator delegates to ElevenLabs (external trust) since TTS is a specialised capability not available in general-purpose LLMs.
*   The Editor runs locally via ComfyUI for watermarking and post-processing, keeping media assets on-premise.

## 4. LACP Interface

### 4.1 Role Mapping

| Domain Role | LACP Role | Justification |
| :--- | :--- | :--- |
| `DIRECTOR` | `ARCHITECT` | Designs and orchestrates the generation pipeline for each job |
| `SCRIPTWRITER` | `DOCUMENTER` | Documents the creative vision as structured prompts and storyboards |
| `NARRATOR` | `BUILDER` | Builds audio assets from text and voice specifications |
| `EDITOR` | `BUILDER` | Builds the final media artefact with watermarking and post-processing |
| `QA` | `REVIEWER` | Reviews all outputs against safety, quality, and consistency criteria |

### 4.2 Trust Level

All agents operate at `INTERNAL` trust except the Narrator, which operates at `EXTERNAL` trust due to its reliance on the ElevenLabs API. External calls are sandboxed and outputs are validated by the QA agent before release.

### 4.3 Context Object

```json
{
  "mission_id": "afx-gen-{job_id}-{timestamp}",
  "objective": "Generate media assets from brief",
  "payload": {
    "brief": "Product launch video for summer collection",
    "persona": "luxe-editorial",
    "modalities": ["image", "video", "audio"],
    "constraints": {
      "maxDurationSeconds": 60,
      "resolution": "1920x1080",
      "budget": 2.50
    },
    "formicContext": {
      "memories": [],
      "genePoolPatterns": [],
      "personaPresets": {}
    }
  }
}
```

*   **`brief`** — Natural language description of the desired output.
*   **`persona`** — One of 9 persona presets that encode style, tone, and visual parameters.
*   **`modalities[]`** — Subset of `["image", "video", "audio"]` to generate.
*   **`constraints`** — Budget cap (USD), resolution, and duration limits.

### 4.4 Feature Flag

```
USE_ARTIFEX_PIPELINE=true
```

*   **`true`** — Activates the full 5-agent pipeline.
*   **`false`** — Falls back to single-model generation with default parameters.

## 5. Tribunal Integration

Artifex defines **5 domain-specific conditions** that trigger a Tribunal review. When triggered, the pipeline pauses and a Tribunal is convened using the standard Commit-Reveal flow (see `../consensus/tribunal.spec.md`).

### 5.1 Trigger Conditions

| # | Condition | Trigger Rule | Jury Composition |
| :--- | :--- | :--- | :--- |
| 1 | **Safety violation** | Content flagged by internal safety scan or SafetyOS | `QA` + `DIRECTOR` + SafetyOS delegate |
| 2 | **Watermark failure** | QA detects watermark degradation below integrity threshold | `QA` + `EDITOR` + `DIRECTOR` |
| 3 | **Persona inconsistency** | Output diverges from persona preset parameters (style/tone drift > 0.3) | `QA` + `SCRIPTWRITER` + `DIRECTOR` |
| 4 | **Budget overrun** | Generation cost exceeds job budget (GPU-intensive operations) | `DIRECTOR` + `QA` + `EDITOR` |
| 5 | **Quality gate** | QA scores output < 0.7 on consistency metrics | Full panel (all 5 agents) |

### 5.2 Tribunal Mechanics

*   **Voting:** Personality-weighted blind voting with sycophancy detection (see `../consensus/sycophancy.spec.md`).
*   **Personality Weighting:** Agents with higher Conscientiousness and lower Agreeableness receive higher effective weight.
*   **Safety Default:** If the Tribunal cannot reach consensus, the job is paused and flagged for human review.
*   **Escalation:** Jobs that fail Tribunal twice are cancelled with full reasoning traces attached.

## 6. Personality System (AAPS)

Each Artifex agent has an **OCEAN personality profile** managed by the Agent Adaptive Personality System (AAPS). Refer to `../consensus/personality.spec.md` for the full specification.

### 6.1 Default Archetypes

Temperature formula: `temp = clamp(O * 0.6 + E * 0.2 - C * 0.15 - N * 0.1, 0.0, 1.0)`

| Agent | Archetype | O | C | E | A | N | Derived Temp |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Director | Commander | 0.65 | 0.80 | 0.55 | 0.50 | 0.40 | ~0.34 |
| Scriptwriter | Storyteller | 0.90 | 0.50 | 0.80 | 0.65 | 0.20 | ~0.61 |
| Narrator | Performer | 0.45 | 0.85 | 0.40 | 0.60 | 0.30 | ~0.19 |
| Editor | Craftsman | 0.35 | 0.90 | 0.30 | 0.45 | 0.50 | ~0.09 |
| QA | Inspector | 0.30 | 0.95 | 0.35 | 0.20 | 0.70 | ~0.04 |

**Calibration Note:** The Scriptwriter has the highest derived temperature (~0.61), reflecting its creative role — high Openness and Extraversion produce more adventurous prompt generation. The QA agent has the lowest (~0.04), ensuring strict, deterministic validation. The Director sits in the middle (~0.34), balancing creative direction with structured orchestration.

### 6.2 Personality Evolution

Personality profiles evolve based on three feedback signals:

1.  **Human Feedback** — When a human rejects or modifies generated media, the originating agent's profile is nudged (e.g., a Scriptwriter producing off-brand prompts has Openness reduced).
2.  **Tribunal Outcomes** — Agents whose votes align with the final verdict receive a small Conscientiousness boost.
3.  **Engagement Metrics** — Post-publication engagement data (views, shares, conversion) feeds back into the Scriptwriter and Director profiles, reinforcing successful creative patterns.

## 7. Watermarking System

Artifex applies three watermarking algorithms to establish content provenance:

| Algorithm | Domain | Robustness | Use Case |
| :--- | :--- | :--- | :--- |
| **LSB** (Least Significant Bit) | Spatial | Moderate — survives light compression | Fast embedding for draft outputs |
| **DCT** (Discrete Cosine Transform) | Frequency | High — survives JPEG re-encoding | Standard watermark for published images |
| **DWT** (Discrete Wavelet Transform) | Multi-resolution | Highest — survives cropping, scaling, and re-encoding | Primary watermark for high-value assets |

All three algorithms embed a provenance payload containing the mission ID, generation timestamp, persona preset, and model version. The QA agent validates watermark integrity before approving any output.

## 8. Cross-Swarm Interactions

| Pattern Flow | Source | Destination | Example |
| :--- | :--- | :--- | :--- |
| Visual guidelines | Artifex | **Provenance** | Product photography composition rules and lighting presets |
| Persona media assets | Artifex | **TalentOS** | Scene generation, profile images, persona visual identity |
| Brand styling context | Provenance | **Artifex** | Colour palettes, typography, and brand voice from luxury listings |
| Persona specifications | TalentOS | **Artifex** | Persona definitions with style parameters for media generation |
| Safety policies | SafetyOS | **Artifex** | Content safety rules and prohibited content patterns |
| Creative patterns | Artifex | **Gene Pool** | Successful prompt templates and generation configurations |

**Propagation Mechanism:** All cross-swarm interactions flow through the Gene Pool's async push/pull protocol. No direct swarm-to-swarm communication occurs — the Gene Pool acts as the sole intermediary, ensuring workspace isolation and audit traceability.
