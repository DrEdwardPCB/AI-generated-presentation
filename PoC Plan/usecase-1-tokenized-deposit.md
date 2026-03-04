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

Color coding: **Orange** = Fireblocks (vendor), **Blue** = Scotiabank vault, **Green** = operational wallet, **Purple** = customer vault account.

### 3.1 Model A — Omnibus + Operational + Client

```mermaid
graph TD
    subgraph Fireblocks["Fireblocks MPC Custody"]
        ROOT["Fireblocks Vendor Custody Root"]
    end

    subgraph Bank["Scotiabank Omnibus"]
        OMNIBUS["Scotiabank Omnibus Vault"]
    end

    subgraph Ops["Bank Operational Wallets"]
        OP1["Treasury Wallet"]
        OP2["Operations Wallet"]
    end

    subgraph Clients["Customer Vault Accounts"]
        C1["Client A Vault Account"]
        C2["Client B Vault Account"]
    end

    ROOT --> OMNIBUS
    OMNIBUS --> OP1
    OMNIBUS --> OP2
    OMNIBUS --> C1
    OMNIBUS --> C2

    style ROOT fill:#f97316,color:#fff
    style OMNIBUS fill:#2563eb,color:#fff
    style OP1 fill:#16a34a,color:#fff
    style OP2 fill:#16a34a,color:#fff
    style C1 fill:#8b5cf6,color:#fff
    style C2 fill:#8b5cf6,color:#fff
```

### 3.2 Model B — Per Client Segregation

```mermaid
graph TD
    subgraph Fireblocks["Fireblocks MPC Custody"]
        ROOT["Fireblocks Vendor Custody Root"]
    end

    subgraph ClientA["Client A"]
        VAULT_A["Scotiabank Vault for Client A"]
        WALLET_A["Client A Vault Account"]
    end

    subgraph ClientB["Client B"]
        VAULT_B["Scotiabank Vault for Client B"]
        WALLET_B["Client B Vault Account"]
    end

    ROOT --> VAULT_A
    ROOT --> VAULT_B
    VAULT_A --> WALLET_A
    VAULT_B --> WALLET_B

    style ROOT fill:#f97316,color:#fff
    style VAULT_A fill:#2563eb,color:#fff
    style VAULT_B fill:#2563eb,color:#fff
    style WALLET_A fill:#8b5cf6,color:#fff
    style WALLET_B fill:#8b5cf6,color:#fff
```

### 3.3 Model C — Per BU Segregation

```mermaid
graph TD
    subgraph Fireblocks["Fireblocks MPC Custody"]
        ROOT["Fireblocks Vendor Custody Root"]
    end

    subgraph BU1["Business Unit 1"]
        VAULT_BU1["BU1 Scotiabank Vault"]
        OPS_BU1["BU1 Operational Wallets"]
        CLIENTS_BU1["BU1 Customer Vault Accounts"]
    end

    subgraph BU2["Business Unit 2"]
        VAULT_BU2["BU2 Scotiabank Vault"]
        OPS_BU2["BU2 Operational Wallets"]
        CLIENTS_BU2["BU2 Customer Vault Accounts"]
    end

    ROOT --> VAULT_BU1
    ROOT --> VAULT_BU2
    VAULT_BU1 --> OPS_BU1
    VAULT_BU1 --> CLIENTS_BU1
    VAULT_BU2 --> OPS_BU2
    VAULT_BU2 --> CLIENTS_BU2

    style ROOT fill:#f97316,color:#fff
    style VAULT_BU1 fill:#2563eb,color:#fff
    style VAULT_BU2 fill:#2563eb,color:#fff
    style OPS_BU1 fill:#16a34a,color:#fff
    style OPS_BU2 fill:#16a34a,color:#fff
    style CLIENTS_BU1 fill:#8b5cf6,color:#fff
    style CLIENTS_BU2 fill:#8b5cf6,color:#fff
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

## 6) Money Movement — Flow Diagram (All Scenarios)

The following flow diagram summarizes **fiat** and **token** movement across all sequence-diagram scenarios (Minting, FX, Transaction, Burning).

```mermaid
flowchart TB
    subgraph Mint["1. Minting"]
        M_CUST["Customer Fiat Acct"]
        M_RES["Reserve Account"]
        M_MIR["Mirror Account A"]
        M_WA["Customer Wallet A"]
        M_CUST -->|Fiat deposit| M_RES
        M_RES -.->|Hold| M_MIR
        M_MIR -.->|Credit mirror| M_MIR
        M_MIR -->|Mint token| M_WA
    end

    subgraph FX["2. FX"]
        F_WA["Wallet A"]
        F_MIR["Mirror A"]
        F_WA -->|Burn USD token| F_WA
        F_WA -->|Mint CAD token| F_WA
        F_MIR -->|Update ledger USD→CAD| F_MIR
    end

    subgraph Tx["3. Transaction"]
        T_WA["Wallet A"]
        T_WB["Wallet B"]
        T_MA["Mirror A"]
        T_MB["Mirror B"]
        T_WA -->|Token transfer| T_WB
        T_MA -->|Debit| T_MA
        T_MB -->|Credit| T_MB
    end

    subgraph Burn["4. Burning"]
        B_WA["Wallet A"]
        B_MIR["Mirror A"]
        B_RES["Reserve Account"]
        B_CUST["Customer Fiat Acct"]
        B_WA -->|Burn token| B_WA
        B_MIR -->|Reduce mirror| B_MIR
        B_RES -->|Release fiat| B_CUST
    end
```

**Legend:**

| Scenario | Fiat movement | Token movement |
|----------|----------------|----------------|
| **1. Minting** | Customer fiat → Reserve; mirror account credited for customer | Token minted to Customer Wallet A |
| **2. FX** | Mirror ledger updated (USD balance down, CAD balance up) | Burn source-currency token; mint target-currency token in same wallet |
| **3. Transaction** | Mirror A debited; Mirror B credited | Token transferred Wallet A → Wallet B |
| **4. Burning** | Reserve releases fiat → Customer fiat account | Token burned in Customer Wallet A |

---

## 7) Goal 1 Test Items (Updated)

| # | Test Item | Expected Result | Business Value |
|---|---|---|---|
| 1.1 | Deploy tokenized deposit contract on private network | Contract deployed and verifiable on private network nodes managed by Fireblocks | Establishes programmable deposit rail |
| 1.2 | Wallet management for tokenized deposit accounts | Required vault accounts and wallets created for client/bank/BU model | Enables scalable account onboarding |
| 1.3 | Mint tokenized deposit from fiat instruction | Mint confirmed; mirror ledger and reserve movement aligned | Proves fiat-token synchronization |
| 1.4 | Transfer tokenized deposit between wallets | On-chain transfer confirmed; sender/receiver mirror accounts updated | Enables internal real-time settlement |
| 1.5 | FX token movement lifecycle | Burn/mint FX flow executed and reconciled with multi-currency ledger | Supports treasury and cross-currency operations |
| 1.6 | Burn/redeem tokenized deposit to fiat | Burn completed; fiat returned through reserve release | Confirms full lifecycle redemption |
| 1.7 | End-to-end reconciliation | On-chain balances and core mirror accounts match with zero breaks | Satisfies control and audit requirements |

