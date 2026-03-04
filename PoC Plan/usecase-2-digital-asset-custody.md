# Use Case 2 - Customer Digital Asset Custody

## 1) Objective

Validate customer custody operating model for stablecoin/CBDC-style assets with clear segregation, governance, and operational controls.

---

## 2) Capability Scope

- Customer wallet onboarding under bank custody model
- Segregated wallet hierarchy by client and by business unit where required
- Inbound asset reception and event streaming
- Outbound transfer with policy checks and approvals
- Reconciliation to core banking mirror records

---

## 3) Custody Operating Models

### Model A - Omnibus anchored
- Bank omnibus vault is the primary anchor.
- Customer wallets and bank operations wallets are children.
- Useful for simpler operations and centralized treasury controls.

### Model B - Dedicated wallet per client
- Each customer has dedicated vault ownership boundary.
- Stronger segregation and cleaner client-level reporting.
- Preferred where legal segregation is required.

### Model C - Dedicated wallet per BU
- Each business unit has dedicated custody domain.
- Supports BU-level PnL, policy, and governance boundaries.
- Can coexist with Model B for large segments.

---

## 4) Sequence - Custody Receive and Mirror

```mermaid
sequenceDiagram
    participant Network as External Network/Testnet
    participant FB as Fireblocks API
    participant Wh as Webhook Receiver
    participant Orch as Orchestration Service
    participant AML as AML Engine
    participant Core as Core Banking
    participant Recon as Reconciliation Service

    Network->>FB: Transfer digital asset to customer wallet
    FB->>Wh: Inbound transaction webhook
    Wh->>Orch: Normalize and enrich event
    Orch->>AML: Screening request
    AML-->>Orch: Cleared or flagged
    Orch->>Core: Update customer mirror account
    Orch->>Recon: Register event for reconciliation
    Recon->>Core: Match on-chain amount vs mirror ledger
```

---

## 5) Goal 2 Test Items (Updated)

| # | Test Item | Expected Result | Business Value |
|---|---|---|---|
| 2.1 | Create customer vault account and wallet | Wallet created with expected governance policy | Establishes onboarding flow for custody |
| 2.2 | Validate custody hierarchy model | Chosen model (omnibus/per-client/per-BU) configured and auditable | Confirms controllable and compliant setup |
| 2.3 | Receive digital asset into customer wallet | Inbound event detected and posted to mirror account | Provides real-time holdings visibility |
| 2.4 | Outbound digital asset transfer with controls | Transfer processed only with policy approvals and AML checks | Demonstrates secure withdrawals |
| 2.5 | On-chain event streaming to bank systems | Webhooks delivered and consumed reliably | Enables operational monitoring and controls |
| 2.6 | Reconciliation with core banking mirror | Reconciliation run produces zero unreconciled breaks | Validates accounting integrity |

---

## 6) Non-Scope Clarification

The following items are out of scope in this use case and this PoC phase:

- Trading APIs and execution flows
- FIX protocol integration
- OMS order lifecycle

