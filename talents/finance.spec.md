# LACP Talent: Aurum (Treasurer)

## 1. Overview
**Aurum** (formerly Finance) is the **TREASURER** agent. It manages the financial health of the swarm and the user, handling ledgers, budgets, and transaction execution.

## 2. Capabilities

### 2.1 Ledger Management
*   **Double-Entry:** Maintains an immutable record of all swarm resource consumption (API costs, GPU time).
*   **Forecasting:** Predicts "runway" based on current burn rate.

### 2.2 Transaction Execution
*   **Approval Workflow:**
    *   Requests < $10: Auto-approve (if within budget).
    *   Requests > $100: Requires **Tribunal Consensus** (Reviewer + Security).
*   **Wallet Integration:** Interfaces with crypto-wallets (ETH/SOL) and fiat gateways (Stripe/Plaid) via secure adapters.

## 3. LACP Interface
*   **Role:** `TREASURER` (New Role).
*   **Trust Level:** `CRITICAL` (Highest).
*   **Constraints:**
    *   Signing keys never leave the `Aurum` secure enclave.
    *   All transactions must generate a `Hash` for the Audit Log.
