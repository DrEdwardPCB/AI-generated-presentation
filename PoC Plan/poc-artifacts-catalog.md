# PoC Artifacts Catalog and Expected Content

## 1) Core Planning Artifacts

### 1.1 `poc-analysis.md`
Expected content:
- Current PoC scope and out-of-scope items
- Updated architecture baseline and ownership boundaries
- Document map to business use case artifacts

### 1.2 `usecase-1-tokenized-deposit.md`
Expected content:
- Business objective and success criteria for tokenized deposits
- Custody hierarchy models (omnibus, per-client, per-BU)
- Sequence diagrams: minting with fiat mirror, FX, transfer, burn
- Goal 1 test matrix with expected outcomes and business value

### 1.3 `usecase-2-digital-asset-custody.md`
Expected content:
- Business objective and success criteria for custody use case
- Custody operating model details and decision points
- Receive/transfer lifecycle and core-banking mirror handling
- Goal 2 test matrix with expected outcomes and business value

---

## 2) Delivery and Execution Artifacts

### 2.1 Environment and infrastructure specification (`infra-spec.md`)
Expected content:
- Environment split by dev/uat/prod control objectives
- Required GCP resources:
  - VPC and subnet segmentation
  - GKE clusters for orchestration and webhook workloads
  - Artifact Registry / container repository
  - Cloud SQL or managed database for operational state and reconciliation
  - Secret management and key handling boundaries
  - GCVE integration points (if required by enterprise landing zone)
  - Logging/monitoring stack and alert policy
- Connectivity model between bank sandbox and Fireblocks sandbox

### 2.2 API contract package (`api-contracts.md`)
Expected content:
- Internal orchestration APIs (mint, transfer, burn, FX request)
- Request/response schemas and idempotency strategy
- Error taxonomy and retry expectations
- Authentication and authorization model

### 2.3 Webhook event contract (`event-contracts.md`)
Expected content:
- Fireblocks webhook event types used in scope
- Event payload schema mapping to bank canonical model
- Delivery guarantees, replay/retry handling, and deduplication keys
- Dead-letter and recovery operating process

### 2.4 Reconciliation specification (`reconciliation-rules.md`)
Expected content:
- Matching logic between on-chain events and core mirror ledger
- Tolerances, cutoff windows, break classification, and exception handling
- End-of-day and intraday controls

### 2.5 Wallet and policy model (`wallet-policy-model.md`)
Expected content:
- Wallet hierarchy options and approval policy templates
- Role-based access controls for operations and approvals
- BU/client segregation rules and migration path between models

### 2.6 Test execution pack (`test-scenarios.md`)
Expected content:
- Detailed test scenarios for Goal 1 and Goal 2
- Preconditions, test steps, expected outcomes, rollback steps
- Negative tests for policy violations, AML holds, and reconciliation breaks

### 2.7 Runbook and operations (`runbook.md`)
Expected content:
- Incident handling workflow for failed mint/transfer/burn
- Service recovery actions for webhook or reconciliation outages
- Daily operational checks and audit evidence list

---

## 3) Recommended Build Order

1. `poc-analysis.md` (scope and boundaries)
2. `usecase-1-tokenized-deposit.md` and `usecase-2-digital-asset-custody.md`
3. `infra-spec.md`
4. `api-contracts.md` and `event-contracts.md`
5. `wallet-policy-model.md`
6. `reconciliation-rules.md`
7. `test-scenarios.md`
8. `runbook.md`

