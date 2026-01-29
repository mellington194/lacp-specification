# LACP Consensus: Sycophancy Detection

## 1. Overview
Detects and mitigates "Groupthink" where agents mimic each other rather than reasoning independently.

## 2. Algorithm

### Step 1: Vectorization
Agent reasoning traces (`reasoning` field in Vote) are converted into high-dimensional vectors (e.g., via text-embedding-3-small).

### Step 2: Similarity Check
Compute Cosine Similarity between the vectors of all agreeing agents.

$$ \text{Similarity}(A, B) = \frac{A \cdot B}{\|A\| \|B\|} $$

### Step 3: Thresholding & Resolution
*   **Threshold:** If $\text{Similarity} > 0.95$:
    *   The output is flagged as **Derivative**.
*   **Resolution:**
    *   Compare historical confidence scores (Pheromone decay).
    *   Discard the vote from the agent with the lower score.
    *   Keep only the "original" thought leader's vote.
