# Use Case 1 - Tokenized Deposit on Private Network

## 1) Objective

Validate end-to-end tokenized deposit lifecycle on a private network managed in Fireblocks GCP sandbox, including wallet management, fiat mirror accounting, and reconciliation.

---

## 2) Target Capability

- Deploy tokenized deposit contract on private network
- Manage vault and wallet hierarchy for bank operations and customers
- Mint, transfer, perform FX movement, and burn tokenized deposits
- Keep fiat and token ledgers synchronized in core banking

---

## 3) Custody Hierarchy Models

```mermaid
graph TD
    ROOT[Fireblocks Vendor Custody Root]

    subgraph "Model A - Omnibus + Operational + Client"
        OMNIBUS[Bank Omnibus Vault]
        OP1[Bank Treasury Wallet]
        OP2[Bank Operations Wallet]
        C1[Client A Wallet]
        C2[Client B Wallet]
        OMNIBUS --> OP1
        OMNIBUS --> OP2
        OMNIBUS --> C1
        OMNIBUS --> C2
    end

    subgraph "Model B - Per Client Segregation"
        CM1[Client A Dedicated Vault]
        CM2[Client B Dedicated Vault]
    end

    subgraph "Model C - Per BU Segregation"
        BU1[BU1 Master Vault]
        BU2[BU2 Master Vault]
    end

    ROOT --> OMNIBUS
    ROOT --> CM1
    ROOT --> CM2
    ROOT --> BU1
    ROOT --> BU2
```

---

## 4) Sequence - Minting with Fiat Mirror

```mermaid
sequenceDiagram
    participant Cust as Customer Fiat Account
    participant Core as Core Banking
    participant Reserve as Core Reserve Account
    participant Mirror as Core Mirror Account
    participant Orch as Orchestration Service
    participant FB as Fireblocks API
    participant Token as Token Contract (Private Net)
    participant Wh as Webhook Receiver
    participant Recon as Reconciliation Service

    Cust->>Core: Deposit fiat instruction (100,000)
    Core->>Reserve: Move fiat to reserve account
    Core->>Mirror: Credit customer mirror account (100,000)
    Core->>Orch: Mint request with customer wallet and amount
    Orch->>FB: Mint tokenized deposit (100,000)
    FB->>Token: mint(customerWallet, 100,000)
    Token-->>FB: Tx confirmed
    FB-->>Orch: Tx submitted/confirmed
    FB->>Wh: Webhook status update
    Wh->>Recon: On-chain mint event
    Recon->>Core: Match mirror account vs token balance
    Recon-->>Core: Reconciliation matched
```

---

## 5) Additional Sequences Requested

### 5.1 FX Scenario (Tokenized Deposit FX Reallocation)

```mermaid
sequenceDiagram
    participant Core as Core Banking
    participant Orch as Orchestration Service
    participant FB as Fireblocks API
    participant Token as Token Contract
    participant Wh as Webhook Receiver
    participant Recon as Reconciliation Service

    Core->>Orch: FX conversion request (USD token to CAD token)
    Orch->>Core: Fetch internal FX rate and approval
    Core-->>Orch: Approved rate and amount
    Orch->>FB: Burn source token amount (USD)
    FB->>Token: burn(clientWallet, usdAmount)
    FB->>FB: Validate policy and signing workflow
    Orch->>FB: Mint target token amount (CAD equivalent)
    FB->>Token: mint(clientWallet, cadAmount)
    FB->>Wh: Burn and mint completion events
    Wh->>Recon: Process FX lifecycle events
    Recon->>Core: Update mirror ledger in both currencies
```

### 5.2 Transaction Scenario (Wallet-to-Wallet Transfer)

```mermaid
sequenceDiagram
    participant ClientA as Client A Wallet
    participant ClientB as Client B Wallet
    participant Orch as Orchestration Service
    participant FB as Fireblocks API
    participant Token as Token Contract
    participant Wh as Webhook Receiver
    participant Recon as Reconciliation Service
    participant Core as Core Banking

    ClientA->>Orch: Transfer request to Client B
    Orch->>FB: Submit transfer transaction
    FB->>Token: transfer(ClientA, ClientB, amount)
    Token-->>FB: Tx confirmed
    FB->>Wh: Transaction completed event
    Wh->>Recon: Transfer event received
    Recon->>Core: Debit A mirror, credit B mirror
    Recon-->>Orch: Transfer reconciled
```

### 5.3 Burning Scenario (Redeem Back to Fiat)

```mermaid
sequenceDiagram
    participant Client as Client Wallet
    participant Core as Core Banking
    participant Reserve as Core Reserve Account
    participant Mirror as Core Mirror Account
    participant Orch as Orchestration Service
    participant FB as Fireblocks API
    participant Token as Token Contract
    participant Wh as Webhook Receiver
    participant Recon as Reconciliation Service

    Client->>Orch: Redeem request (burn token for fiat)
    Orch->>FB: Burn tokenized deposit
    FB->>Token: burn(clientWallet, amount)
    Token-->>FB: Burn confirmed
    FB->>Wh: Burn completion event
    Wh->>Recon: Burn event processed
    Recon->>Core: Reduce mirror account amount
    Core->>Reserve: Release fiat from reserve
    Core-->>Client: Fiat credited to customer account
```

---

## 6) Goal 1 Test Items (Updated)

| # | Test Item | Expected Result | Business Value |
|---|---|---|---|
| 1.1 | Deploy tokenized deposit contract on private network | Contract deployed and verifiable on private network nodes managed by Fireblocks | Establishes programmable deposit rail |
| 1.2 | Wallet management for tokenized deposit accounts | Required vault accounts and wallets created for client/bank/BU model | Enables scalable account onboarding |
| 1.3 | Mint tokenized deposit from fiat instruction | Mint confirmed; mirror ledger and reserve movement aligned | Proves fiat-token synchronization |
| 1.4 | Transfer tokenized deposit between wallets | On-chain transfer confirmed; sender/receiver mirror accounts updated | Enables internal real-time settlement |
| 1.5 | FX token movement lifecycle | Burn/mint FX flow executed and reconciled with multi-currency ledger | Supports treasury and cross-currency operations |
| 1.6 | Burn/redeem tokenized deposit to fiat | Burn completed; fiat returned through reserve release | Confirms full lifecycle redemption |
| 1.7 | End-to-end reconciliation | On-chain balances and core mirror accounts match with zero breaks | Satisfies control and audit requirements |

