# LACP Security: PII Redaction

## 1. Overview
Automated hygiene filters run at the Router level to prevent sensitive data leakage.

## 2. Named Entity Recognition (NER)
Before any message leaves the trust boundary (e.g., from an Internal Agent to an External API):
*   **Entities Targeted:**
    *   Credit Card Numbers
    *   Social Security Numbers (SSN) / Tax IDs
    *   Email Addresses (unless whitelist matched)
    *   Phone Numbers
*   **Action:** Redaction via replacement character (e.g., `[REDACTED-SSN]`).

## 3. Implementation
The scanner operates on the raw text payload of the `Context Object` before it is serialized for transport.
