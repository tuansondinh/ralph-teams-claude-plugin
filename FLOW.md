# Execution Flow

```mermaid
flowchart TD
    classDef user fill:#ffdfba,stroke:#ffb347,stroke-width:2px,color:#333
    classDef orch fill:#bae1ff,stroke:#5facff,stroke-width:2px,color:#333
    classDef agent fill:#baffc9,stroke:#42d669,stroke-width:2px,color:#333
    classDef doc fill:#ffffba,stroke:#e6e65a,stroke-width:2px,color:#333
    classDef cmd fill:#f3e8ff,stroke:#c084fc,stroke-width:2px,color:#333

    U((User)):::user

    CmdPlan[/teams:plan]:::cmd
    CmdRun[/teams:run]:::cmd
    CmdVerify[/teams:verify]:::cmd

    O[Orchestrator]:::orch
    P[.build/PLAN.md]:::doc

    subgraph Build["Sequential Build"]
        direction TB
        B1[Sonnet Builder — Task 1]:::agent
        B2[Sonnet Builder — Task 2]:::agent
        BN[Sonnet Builder — Task N]:::agent
        B1 --> B2 --> BN
    end

    R[Opus Reviewer]:::agent
    REV[.build/REVIEW.md]:::doc
    BF[Sonnet Builder — Fixes]:::agent
    V[Manual Verify with User]:::orch

    U --> CmdPlan
    U --> CmdRun
    U --> CmdVerify

    CmdPlan --> |"1. Discuss + Approve"| O
    CmdRun --> |"1. Load existing"| O

    O -. "2. Creates / Reads" .-> P
    O -- "3. Spawns per task" --> Build

    Build -- "4. All tasks done" --> O
    O -- "5. Spawns reviewer" --> R
    R -. "Writes" .-> REV
    R -- "6. Review done" --> O
    O -- "7. If fixes needed" --> BF
    BF -- "8. Fixes applied" --> O
    O -- "9. Done" --> U

    CmdVerify --> V
    V -. "Reads" .-> P
    V -. "Reads" .-> REV
    V -- "Results" --> U
```
