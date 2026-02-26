# LACP Consensus: Sycophancy Detection

## 1. Overview
Detects and mitigates "Groupthink" where agents mimic each other rather than reasoning independently. This specification defines two complementary detection mechanisms: **pairwise similarity** (threshold-based) and **cluster detection** (identifies emergent agreement clusters that may indicate systemic sycophancy).

## 2. Pairwise Similarity Detection

### 2.1 Vectorisation
Agent reasoning traces (`reasoning` field in Vote) are converted into high-dimensional vectors using a text embedding model.

### 2.2 Similarity Computation
Compute Cosine Similarity between the vectors of all agreeing agents.

$$ \text{Similarity}(A, B) = \frac{A \cdot B}{\|A\| \|B\|} $$

### 2.3 Configurable Thresholds
Sycophancy detection uses a three-zone threshold system. Default thresholds are provided; implementations MAY override these per-swarm.

| Zone | Classification | Action |
|------|----------------|--------|
| **Safe** | Independent reasoning | No action. Votes are counted normally. |
| **Warning** | Potentially derivative | A `SYCOPHANCY_WARNING` event is emitted. Votes are counted but flagged for audit. |
| **Derivative** | Derivative reasoning detected | The vote from the agent with the lower `tribunalVoteWeight` is discarded. |

Implementations MUST validate that `warningThreshold < derivativeThreshold`. Both values MUST fall within `[0.50, 0.99]`.

### 2.4 Pairwise Resolution
When a pair exceeds the derivative threshold:

1.  Compare the `tribunalVoteWeight` of both agents (derived from OCEAN Conscientiousness; see `consensus/personality.spec.md`).
2.  Discard the vote from the agent with the **lower** weight.
3.  If weights are equal, discard the vote from the agent with the **lower historical accuracy**.
4.  If historical accuracy is also equal, discard the vote from the agent that submitted its commit hash **later** (the follower, not the leader).

## 3. Cluster Detection

Pairwise similarity catches cases where two agents produce near-identical reasoning. Cluster detection addresses a subtler failure mode: when **three or more** agents converge on near-identical reasoning without any single pair exceeding the derivative threshold.

### 3.1 Algorithm

#### Step 1: Compute Similarity Matrix
For `n` voting agents, compute the full `n x n` pairwise cosine similarity matrix.

#### Step 2: Identify Clusters
Apply single-linkage agglomerative clustering with a distance threshold of `(1 - warningThreshold)`. Two agents are linked if their similarity exceeds `warningThreshold`.

Extract all connected components (clusters). A cluster is flagged as **sycophantic** if it meets **both** conditions:
*   **Size:** The cluster contains `>= minClusterSize` agents.
*   **Density:** The mean intra-cluster similarity exceeds `warningThreshold`.

#### Step 3: Resolve Sycophantic Clusters
When a sycophantic cluster is detected:

1.  **Elect a representative.** Within the cluster, select the agent with the highest `tribunalVoteWeight`.
2.  **Discard follower votes.** All other agents in the cluster have their votes discarded.
3.  **Emit event.** A `SYCOPHANCY_CLUSTER_DETECTED` event is emitted.
4.  **Re-tally.** The Tribunal re-tallies votes using only the representative's vote from the cluster plus all votes from agents outside the cluster.

### 3.2 Rapid Convergence Detection

Cluster detection also monitors temporal patterns across sequential Tribunal convocations. If the same cluster of agents agrees with high mean similarity across multiple consecutive Tribunals, a `SYCOPHANCY_RAPID_CONVERGENCE` event is emitted.

The Coordinator SHOULD respond by:
1.  Shuffling agent execution order for the next mission.
2.  Reducing shared context injection.
3.  Temporarily increasing the `temperature` of agents in the convergent cluster.

## 4. Integration with Tribunal Flow

Sycophancy detection executes during **Phase 4 (Tally)** of the Commit-Reveal flow (see `consensus/tribunal.spec.md`).

### 4.1 Execution Order
1.  **Reveal phase completes.** All votes and reasoning traces are available.
2.  **Vectorise.** Embed all reasoning traces.
3.  **Pairwise detection.** Check all pairs against the derivative threshold. Discard derivative votes.
4.  **Cluster detection.** On the remaining votes, run cluster analysis. Discard follower votes from any sycophantic clusters.
5.  **Tally.** Compute the weighted score using only surviving votes.

Pairwise detection runs first because it is cheaper (O(n^2) comparisons, no clustering). Cluster detection runs on the remaining votes.

### 4.2 Personality Interaction
Agents with high Agreeableness are statistically more likely to appear in sycophantic clusters. The personality system accounts for this by assigning lower `tribunalVoteWeight` to agreeable agents, which means they are more likely to be the follower (discarded) rather than the representative (retained) when a cluster is resolved.

### 4.3 Audit Trail
All sycophancy events are recorded in the Tribunal audit log.

## 5. Tuning Guidelines

### 5.1 Threshold Selection
*   **High-stakes swarms**: Lower thresholds to catch subtle agreement patterns early.
*   **Creative swarms**: Higher thresholds to allow natural creative convergence without false positives.
*   Implementations should calibrate thresholds for their specific domain.

### 5.2 Cluster Size
The `minClusterSize` parameter controls sensitivity. Setting it too low generates excessive false positives. Setting it too high may miss groupthink in smaller juries.

### 5.3 Embedding Model
The choice of embedding model affects similarity scores. Implementations SHOULD benchmark their chosen model against a labelled dataset of known-sycophantic and known-independent reasoning pairs to calibrate thresholds.
