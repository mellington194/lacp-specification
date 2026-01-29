# LACP Layer 3: Tribunal Consensus (Detailed)

## 1. The Tribunal Logic
The Tribunal is the **Governance Engine** for the swarm. It prevents "Single Point of Failure" in decision making.

### 2. The Commit-Reveal Flow (Step-by-Step)

#### Phase 1: Proposal
*   **Agent A** proposes: "Post this tweet."
*   **Router** selects Jury: `[REVIEWER, SECURITY, SAFETY_OS]`

#### Phase 2: Commit (Blind Vote)
*   **Reviewer** thinks "Approves". Generates Salt `xyz123`.
    *   Sends Hash: `SHA256("APPROVE" + "xyz123")` -> `a1b2c3...`
*   **Security** thinks "Reject". Generates Salt `abc987`.
    *   Sends Hash: `SHA256("REJECT" + "abc987")` -> `d4e5f6...`
*   *State:* The Router has hashes, but doesn't know the votes. No agent knows what the other voted.

#### Phase 3: Reveal
*   **Router** signals: "All votes in. Open keys."
*   **Reviewer** sends: `{"vote": "APPROVE", "salt": "xyz123"}`.
*   **Security** sends: `{"vote": "REJECT", "salt": "abc987"}`.

#### Phase 4: Tally & Sycophancy Check
*   **Router** calculates Vector Similarity.
*   **If** `Similarity(Reviewer, Security) > 0.95`:
    *   Discard the vote of the lower-ranked agent.
*   **Final Count:** 1 APPROVE vs 1 REJECT.
    *   **Result:** `REJECT` (Safety Default).

## 3. Mathematical Model (Sycophancy)

$$ \text{SycophancyScore} = \max_{i \neq j} \left( \frac{\mathbf{v}_i \cdot \mathbf{v}_j}{\|\mathbf{v}_i\| \|\mathbf{v}_j\|} \right) $$

Where $\mathbf{v}_i$ is the embedding vector of Agent $i$'s `reasoning` text.

*   **Safe Zone:** Score < 0.85
*   **Warning:** 0.85 - 0.95
*   **Rejection:** > 0.95 (Identical reasoning detected).