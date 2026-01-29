# Programme RAID Log (LACP Specification)

**R**isks, **A**ssumptions, **I**ssues, **D**ependencies.

## Risks
*   **R-LACP-001:** Tribunal latency may be too high for real-time social interactions (Social Bot).
    *   *Mitigation:* Implement "Fast Track" voting for low-risk actions.
*   **R-LACP-002:** "Sycophancy Detection" via vector similarity might produce false positives for standardized legal boilerplate.
    *   *Mitigation:* Tune thresholds specifically for the `SAFETY` role.

## Assumptions
*   **A-LACP-001:** All agents can support gRPC or HTTP/2 for the Transport Layer.
*   **A-LACP-002:** The "Context Object" size will stay under 1MB to avoid network bottlenecks.

## Dependencies
*   **D-LACP-001:** Requires `text-embedding-3-small` (or equivalent) for the Sycophancy Detector.
