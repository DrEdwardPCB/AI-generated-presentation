# Posting Model Diagrams (US -> Canada)

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

## 2. Scenario: Async Path (FPS Timeout -> Kafka Success)
*FPS API times out, EPICS waits for Kafka, then proceeds successfully.*

```mermaid
sequenceDiagram
    participant FTM
    participant EPICS
    participant FPS as FPS (DDA/SG)
    participant Finacle_IBD
    participant Kafka

    FTM->>EPICS: Request Leg 2 (IBD, DDA, SG)
    EPICS->>FPS: Post DDA/SG Legs
    FPS-->>EPICS: Timeout / Error
    EPICS-->>FTM: 202 Accepted
    
    Note right of EPICS: Wait for Kafka (Max 2 mins)
    Kafka-->>EPICS: Success Event Received
    
    Note right of EPICS: Proceed to Step 6 Logic
    EPICS->>Finacle_IBD: Post IBD Legs
    Finacle_IBD-->>EPICS: Success
    
    EPICS->>Kafka: Send Success Event to FTM
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

## 4. Scenario: Async Failure (Kafka Timeout or Fail Event)
*FPS API times out, and either Kafka never responds or sends a fail event.*

```mermaid
sequenceDiagram
    participant FTM
    participant EPICS
    participant FPS as FPS (DDA/SG)
    participant Kafka

    FTM->>EPICS: Request Leg 2 (IBD, DDA, SG)
    EPICS->>FPS: Post DDA/SG Legs
    FPS-->>EPICS: Timeout / Error
    EPICS-->>FTM: 202 Accepted
    
    loop Wait Max 2 mins
        EPICS->>EPICS: Check for Kafka
    end
    
    alt Kafka Timeout
        Note right of EPICS: No event received > 2 mins
        EPICS->>Kafka: Send Error/Fail Event to FTM
    else Kafka Fail Event
        Kafka-->>EPICS: Fail Event Received
        EPICS->>Kafka: Send Fail Event to FTM
    end
```

## 5. Scenario: Async Partial Success (FPS Timeout -> Kafka Success -> IBD Fail)
*FPS API times out, Kafka confirms success, but Finacle IBD fails. EPICS reports partial success via Kafka.*

```mermaid
sequenceDiagram
    participant FTM
    participant EPICS
    participant FPS as FPS (DDA/SG)
    participant Finacle_IBD
    participant Kafka

    FTM->>EPICS: Request Leg 2 (IBD, DDA, SG)
    EPICS->>FPS: Post DDA/SG Legs
    FPS-->>EPICS: Timeout / Error
    EPICS-->>FTM: 202 Accepted
    
    Note right of EPICS: Wait for Kafka (Max 2 mins)
    Kafka-->>EPICS: Success Event Received
    
    Note right of EPICS: Proceed to Step 6 Logic
    EPICS->>Finacle_IBD: Post IBD Legs
    Finacle_IBD-->>EPICS: Fail
    
    Note right of EPICS: No Reversal Triggered
    EPICS->>Kafka: Send Partial Success Event (FPS OK / IBD Fail) to FTM
    
    Note over FTM: FTM decides next step\n(Reverse or Retry)
```

## 6. Logic Flowchart

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
    
    %% Async Path
    FPSResp -- Timeout/Fail --> AsyncResp["Respond 202 Accepted to FTM"]
    AsyncResp --> WaitKafka["Wait for Kafka Event\n(Max 2 mins)"]
    WaitKafka --> KafkaCheck{"Event Received?"}
    
    %% Kafka Timeout
    KafkaCheck -- No/Timeout --> SendKafkaFail["Send Error Event to FTM"]
    SendKafkaFail --> EndFail
    
    %% Kafka Fail
    KafkaCheck -- Fail Event --> SendKafkaFail
    
    %% Kafka Success
    KafkaCheck -- Success Event --> FinIBDAsync["EPICS: Call Finacle IBD"]
    FinIBDAsync --> IBDAsyncResp{"IBD Response?"}
    
    IBDAsyncResp -- OK --> SendKafkaSuccess["Send Success Event to FTM"]
    SendKafkaSuccess --> EndSuccess
    
    IBDAsyncResp -- Fail --> SendKafkaPartial["Send IBD Fail / FPS Success Event to FTM"]
    SendKafkaPartial --> EndPartial
```

## 7. Accounting T-Charts (Sequence of Events)

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
