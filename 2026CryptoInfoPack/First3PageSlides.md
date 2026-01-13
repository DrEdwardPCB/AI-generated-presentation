Perfect — this is a much better, more digestible starting point 👍
Below is a clean, executive-friendly storyboard for the first 3 pages only, written as a Mural-style Mermaid JS board in Markdown, focusing on conceptual understanding, not technology yet.

The tone is:
	•	Non-technical
	•	Narrative
	•	Builds intuition before blockchain jargon
	•	Answers “why this exists” before “how it works”

⸻

Slide 1 — What does a bank actually do today? (and why trust exists)

Story goal

Help everyone align on:
	•	What banks really do
	•	Why bookkeeping & trust are central
	•	Why complexity exists even without crypto

flowchart LR
  subgraph Bank["What a bank does"]
    A1["Earn Interest Income\n• loans\n• mortgages\n• credit"]
    A2["Earn Non-Interest Income\n• fees\n• payments\n• FX spreads"]
    A3["Trading Income\n• FX\n• securities\n• derivatives"]
  end

  subgraph Operations["Core banking operations"]
    O1["Transaction processing\n(payments, transfers)"]
    O2["Bookkeeping\n(credits & debits)"]
    O3["Inter-bank settlement\n(clearing & settlement)"]
  end

  subgraph Trust["Why people trust banks"]
    T1["Central ledger\n(single source of truth)"]
    T2["Regulation & audits"]
    T3["Capital & liquidity buffers"]
    T4["Operational controls\n(separation of duties)"]
  end

  Bank --> Operations
  Operations --> Trust

  subgraph Pain["Why it is hard"]
    P1["Multiple banks involved"]
    P2["Each bank keeps its own books"]
    P3["Reconciliation needed\nbetween institutions"]
    P4["Delays (T+1, T+2)\n+ operational cost"]
  end

  Operations --> Pain

Key takeaway (spoken, not technical)

Banks are trusted bookkeepers.
Money movement is really book updates across institutions, and trust comes from controls, regulation, and reconciliation — not magic.

⸻

Slide 2 — Crypto currencies: the idea of a shared ledger

Story goal

Introduce shared ledger thinking without saying “blockchain” first.

flowchart LR
  subgraph Traditional["Traditional banking model"]
    B1["Bank A Ledger"]
    B2["Bank B Ledger"]
    B3["Bank C Ledger"]
  end

  subgraph Reconciliation["What must happen"]
    R1["Messages sent between banks"]
    R2["Each bank updates own books"]
    R3["Reconciliation & dispute handling"]
  end

  Traditional --> Reconciliation

  subgraph Shared["Shared ledger concept"]
    S1["One shared record\nseen by all participants"]
    S2["Same transaction state\nfor everyone"]
    S3["No need to reconcile\nmultiple versions"]
  end

  Reconciliation --> Shared

Introduce crypto carefully

flowchart TB
  C1["Crypto currencies introduce\nshared ledgers"]
  C2["Instead of trusting one bank,\nparticipants trust the rules"]
  C3["Rules decide:\n• what is valid\n• who can update\n• when it is final"]

  C1 --> C2 --> C3

Key terminology (light, no deep dive)
	•	Ledger: a record of balances and changes
	•	Shared ledger: many parties see the same record
	•	Validation: how participants agree something is correct

We will explain how validation and synchronization work later.

⸻

Slide 3 — Crypto: what does it replace (and what it does NOT)

Story goal

Avoid fear. Be explicit:
	•	Crypto does not replace everything
	•	It replaces specific roles in the system

flowchart LR
  subgraph BankRoles["Roles in traditional banking"]
    R1["Ledger keeping\n(source of truth)"]
    R2["Settlement & clearing"]
    R3["Trust broker\n('this transaction is valid')"]
    R4["Custody & control"]
    R5["Customer relationship"]
    R6["Risk, compliance, regulation"]
  end

  subgraph CryptoReplace["Crypto can replace or change"]
    C1["Shared ledger\ninstead of single bank ledger"]
    C2["Rule-based validation\ninstead of bilateral trust"]
    C3["Atomic settlement\ninstead of delayed clearing"]
  end

  subgraph CryptoNot["Crypto does NOT replace"]
    N1["Regulation & compliance"]
    N2["Risk management"]
    N3["Customer onboarding"]
    N4["Product design"]
    N5["Legal accountability"]
  end

  BankRoles --> CryptoReplace
  BankRoles --> CryptoNot

Framing sentence for leadership

Crypto is not trying to be a bank.
It replaces how trust and record-keeping are achieved, not why banks exist.

⸻

Why this structure works for your boss’s concern
	•	✅ No blockchain mechanics yet
	•	✅ Uses bank language first
	•	✅ Builds intuition before terminology
	•	✅ Sets up future slides naturally:
	•	“If shared ledgers exist → how do they stay in sync?”
	•	“If trust is rule-based → who writes the rules?”

⸻

Next step (when you’re ready)

I can:
	•	Rewrite this as speaker notes
	•	Turn this into actual slide text
	•	Or design Slide 4 as:

“How do shared ledgers agree? (Validation & synchronization — simplified)”

Just tell me which direction you want 👍