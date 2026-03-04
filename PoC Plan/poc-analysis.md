# Scotiabank x Fireblocks PoC - Analysis Overview

## 1) Scope Update (2026-03-04)

This PoC is now focused on:

- Tokenized deposit on a private network fully managed in the Fireblocks GCP sandbox
- Wallet management and custody hierarchy validation
- Customer digital asset custody (stablecoin/CBDC holding model)
- Reconciliation between on-chain records and core banking mirror ledgers

Out of scope for this PoC phase:

- Trading API use cases
- FIX-to-REST integration
- OMS trading flows

---

## 2) Architecture Baseline

### 2.1 High-Level Architecture (Updated)

```mermaid
graph TB
    subgraph "Scotiabank GCP Sandbox"
        direction TB

        subgraph "Bank Core Systems"
            CORE[Core Banking System]
            AML[AML / Compliance Engine]
            RECON[Reconciliation Engine]
        end

        subgraph "Bank Integration Layer"
            ORCH[Orchestration Service<br/>API Gateway]
            WH_RECV[Webhook Receiver<br/>Event Processor]
            RECON_SVC[Reconciliation Service]
        end

        CORE <-->|Account and Balance| ORCH
        AML -->|Screening Results| ORCH
        ORCH <-->|State Sync| RECON_SVC
        RECON_SVC <-->|Ledger Match| RECON
        RECON <-->|Balance Check| CORE
        WH_RECV -->|On-chain Events| RECON_SVC
    end

    subgraph "Fireblocks GCP Sandbox (Vendor Managed)"
        direction TB

        subgraph "Fireblocks Workspace"
            FB_API[Fireblocks REST API<br/>api.fireblocks.io]
            FB_VAULT[Vault Management<br/>MPC Co-Signer]
            FB_TOKEN[Tokenization Engine]
            FB_SC[Smart Contract Management]
            FB_WH[Webhook Service v2]
        end

        subgraph "Private Network (Fireblocks Custody)"
            PN1[Private Node 1]
            PN2[Private Node 2]
            PN3[Private Node 3]
            ORACLE[Data Oracle Contract]
        end

        FB_API --> FB_VAULT
        FB_API --> FB_TOKEN
        FB_API --> FB_SC
        FB_WH --> FB_API

        PN1 <--> PN2
        PN2 <--> PN3
        PN3 <--> PN1
        ORACLE -.->|Deployed on| PN1
    end

    ORCH <-->|REST API + JWT| FB_API
    FB_WH -->|HTTPS Push Events| WH_RECV
    FB_SC <-->|Contract Deploy and Call| PN1

    style CORE fill:#2563eb,color:#fff
    style AML fill:#2563eb,color:#fff
    style RECON fill:#2563eb,color:#fff
    style ORCH fill:#7c3aed,color:#fff
    style WH_RECV fill:#7c3aed,color:#fff
    style RECON_SVC fill:#7c3aed,color:#fff
    style FB_API fill:#f97316,color:#fff
    style FB_VAULT fill:#f97316,color:#fff
    style FB_TOKEN fill:#f97316,color:#fff
    style FB_SC fill:#f97316,color:#fff
    style FB_WH fill:#f97316,color:#fff
    style PN1 fill:#16a34a,color:#fff
    style PN2 fill:#16a34a,color:#fff
    style PN3 fill:#16a34a,color:#fff
```

---

## 3) Custody Hierarchy Expansion

Custody hierarchy now supports multiple operating models, not only one omnibus pattern:

- Model A: Omnibus wallet + bank operational wallets + client wallets
- Model B: Per-client wallet segregation (one wallet per client)
- Model C: Business-unit wallet segregation (one wallet per BU)

Detailed diagrams and operating rules are in:

- `AI-generated-presentation/PoC Plan/usecase-1-tokenized-deposit.md`
- `AI-generated-presentation/PoC Plan/usecase-2-digital-asset-custody.md`

---

## 4) Test Goal Scope (Revised)

Only Goal 1 and Goal 2 remain in this phase:

- Goal 1: Tokenized deposit on private network, including wallet management
- Goal 2: Customer digital asset custody model (stablecoin/CBDC holding)

Goal 3 (trading) is removed from this document set.

---

## 5) Artifact Split by Business Use Case

This overview document is intentionally concise. Detailed content is split into separate artifacts:

- `AI-generated-presentation/PoC Plan/usecase-1-tokenized-deposit.md`
- `AI-generated-presentation/PoC Plan/usecase-2-digital-asset-custody.md`
- `AI-generated-presentation/PoC Plan/poc-artifacts-catalog.md`
