# Posting Model Diagrams (Interim) - US -> Canada

## 1. Scenario: Happy Path (Synchronous Success)
*Everything succeeds immediately via API.*

```mermaid
sequenceDiagram
    participant FTM
    participant EPICS
    participant Finacle_US
    participant FPS as FPS (DDA/SG)
    participant Finacle_IBD

    Note over FTM, Finacle_US: Phase 1: Finacle US Legs
    FTM->>FTM: Sanction Check
    FTM->>EPICS: Request Leg 1 (Finacle US)
    EPICS->>Finacle_US: Post Debit/Credit
    Finacle_US-->>EPICS: Success
    EPICS-->>FTM: API Response (Completed)

    Note over FTM, Finacle_IBD: Phase 2: Cross Border Legs
    FTM->>EPICS: Request Leg 2 (IBD, DDA, SG)
    
    Note right of EPICS: Step 5: FPS (DDA/SG)
    EPICS->>FPS: Post DDA/SG Legs
    FPS-->>EPICS: 200 OK (Success)
    
    Note right of EPICS: Step 6: Finacle IBD
    EPICS->>Finacle_IBD: Post IBD Legs
    Finacle_IBD-->>EPICS: Success
    
    EPICS-->>FTM: 200 Success
```

## 2. Scenario: Async Path (FPS Timeout -> Inquiry Success)
*FPS API times out, EPICS waits 5 minutes, calls Inquiry endpoint, then proceeds successfully.*

```mermaid
sequenceDiagram
    participant FTM
    participant EPICS
    participant FPS as FPS (DDA/SG)
    participant Finacle_IBD

    FTM->>EPICS: Request Leg 2 (IBD, DDA, SG)
    EPICS->>FPS: Post DDA/SG Legs
    FPS-->>EPICS: Timeout
    EPICS-->>FTM: 202 Accepted
    
    Note right of EPICS: Wait 5 minutes
    EPICS->>FPS: Call Inquiry Endpoint
    FPS-->>EPICS: Success Response
    
    Note right of EPICS: Proceed to Step 6 Logic
    EPICS->>Finacle_IBD: Post IBD Legs
    Finacle_IBD-->>EPICS: Success
    
    EPICS->>FTM: Send Success Notification (e.g. Callback/Event)
```

## 3. Scenario: Synchronous Error (Finacle IBD Fail - Partial Success)
*FPS succeeds, but Finacle IBD fails. EPICS reports partial success (IBD Fail) to FTM without reversing.*

```mermaid
sequenceDiagram
    participant FTM
    participant EPICS
    participant FPS as FPS (DDA/SG)
    participant Finacle_IBD

    FTM->>EPICS: Request Leg 2 (IBD, DDA, SG)
    EPICS->>FPS: Post DDA/SG Legs
    FPS-->>EPICS: 200 OK
    
    EPICS->>Finacle_IBD: Post IBD Legs
    Finacle_IBD-->>EPICS: Fail
    
    Note right of EPICS: No Reversal Triggered
    EPICS-->>FTM: Response: IBD Fail / FPS Success
    
    Note over FTM: FTM decides next step\n(Reverse or Retry)
```

## 4. Scenario: Async Failure (Inquiry Timeout or Fail)
*FPS API times out, EPICS waits 5 minutes, calls Inquiry, and Inquiry fails or times out.*

```mermaid
sequenceDiagram
    participant FTM
    participant EPICS
    participant FPS as FPS (DDA/SG)

    FTM->>EPICS: Request Leg 2 (IBD, DDA, SG)
    EPICS->>FPS: Post DDA/SG Legs
    FPS-->>EPICS: Timeout
    EPICS-->>FTM: 202 Accepted
    
    Note right of EPICS: Wait 5 minutes
    EPICS->>FPS: Call Inquiry Endpoint
    
    alt Inquiry Timeout / Fail
        FPS-->>EPICS: Error / Timeout
        EPICS->>FTM: Send Error/Fail Notification
    end
```

## 5. Scenario: Async Partial Success (FPS Timeout -> Inquiry Success -> IBD Fail)
*FPS API times out, Inquiry confirms success, but Finacle IBD fails. EPICS reports partial success.*

```mermaid
sequenceDiagram
    participant FTM
    participant EPICS
    participant FPS as FPS (DDA/SG)
    participant Finacle_IBD

    FTM->>EPICS: Request Leg 2 (IBD, DDA, SG)
    EPICS->>FPS: Post DDA/SG Legs
    FPS-->>EPICS: Timeout
    EPICS-->>FTM: 202 Accepted
    
    Note right of EPICS: Wait 5 minutes
    EPICS->>FPS: Call Inquiry Endpoint
    FPS-->>EPICS: Success Response
    
    Note right of EPICS: Proceed to Step 6 Logic
    EPICS->>Finacle_IBD: Post IBD Legs
    Finacle_IBD-->>EPICS: Fail
    
    Note right of EPICS: No Reversal Triggered
    EPICS->>FTM: Send Partial Success Notification (FPS OK / IBD Fail)
    
    Note over FTM: FTM decides next step\n(Reverse or Retry)
```

## 6. Scenario: FPS Error (Immediate Fail)
*FPS API returns an explicit Error (not Timeout). EPICS fails immediately.*

```mermaid
sequenceDiagram
    participant FTM
    participant EPICS
    participant FPS as FPS (DDA/SG)

    FTM->>EPICS: Request Leg 2 (IBD, DDA, SG)
    EPICS->>FPS: Post DDA/SG Legs
    FPS-->>EPICS: Error (500/400 etc.)
    EPICS-->>FTM: 500 Fail
```

## 7. Logic Flowchart (Interim)

```mermaid
flowchart TD
    Start([Start]) --> Sanction["FTM: Sanction Check"]
    Sanction --> Leg1["FTM: Send Leg 1 Request\n(Finacle US)"]
    Leg1 --> EPICS1["EPICS: Process Leg 1"]
    EPICS1 --> Leg1Success{"Success?"}
    Leg1Success -- No --> EndFail(["End: Fail"])
    Leg1Success -- Yes --> Leg2["FTM: Send Leg 2 Request\n(IBD, DDA, SG)"]
    
    Leg2 --> FPSCall["EPICS: Call FPS API\n(DDA/SG Legs)"]
    FPSCall --> FPSResp{"FPS API Response?"}
    
    %% Happy Path
    FPSResp -- OK --> FinIBD["EPICS: Call Finacle IBD"]
    FinIBD --> IBDResp{"IBD Response?"}
    IBDResp -- OK --> SuccessResp["Respond 200 Success to FTM"]
    SuccessResp --> EndSuccess(["End: Success"])
    
    %% Partial Success Path (Synchronous)
    IBDResp -- Fail --> PartialFail["Respond IBD Fail / FPS Success to FTM"]
    PartialFail --> EndPartial(["End: Partial Success\n(FTM Decision)"])
    
    %% Error Path (Immediate)
    FPSResp -- Error --> ErrorResp["Respond 500 Fail to FTM"]
    ErrorResp --> EndFail
    
    %% Async Path (Timeout)
    FPSResp -- Timeout --> AsyncResp["Respond 202 Accepted to FTM"]
    AsyncResp --> Wait5Min["Wait 5 Minutes"]
    Wait5Min --> InquiryCall["Call FPS Inquiry Endpoint"]
    InquiryCall --> InquiryResp{"Inquiry Response?"}
    
    %% Inquiry Fail
    InquiryResp -- Fail/Timeout --> SendAsyncFail["Send Error Notification to FTM"]
    SendAsyncFail --> EndFail
    
    %% Inquiry Success
    InquiryResp -- Success --> FinIBDAsync["EPICS: Call Finacle IBD"]
    FinIBDAsync --> IBDAsyncResp{"IBD Response?"}
    
    IBDAsyncResp -- OK --> SendAsyncSuccess["Send Success Notification to FTM"]
    SendAsyncSuccess --> EndSuccess
    
    IBDAsyncResp -- Fail --> SendAsyncPartial["Send IBD Fail / FPS Success Notification to FTM"]
    SendAsyncPartial --> EndPartial
```

## 8. Accounting T-Charts (Sequence of Events)

### Phase 1: Finacle US (Step 2)

| Finacle US | |
| :--- | :--- |
| **Debit** | **Credit** |
| Customer Account | |
| | ICA Account (GUS.IBD) |

### Phase 2: Cross Border - First Half (FPS - DDA & SG) (Step 4 & 5)

| DDA System | |
| :--- | :--- |
| **Debit** | **Credit** |
| DDA Pool Account | |
| | Customer Account (Canada) |
| | DDA Pool Account |

| SG System | |
| :--- | :--- |
| **Debit** | **Credit** |
| SG FNCL Offset | |
| ICA(HOU) | |
| | Fin Offset |

### Phase 2: Cross Border - Second Half (Finacle IBD) (Step 4 & 6)

| Finacle IBD | |
| :--- | :--- |
| **Debit** | **Credit** |
| ICA(HOU) | |
| | Suspense Account |
