# Default Execution Flow

This diagram illustrates how the plugin operates from top to bottom. It shows how the two primary commands (`/teams:plan` and `/teams:run`) feed into the Orchestrator, which then manages the execution loop.

```mermaid
flowchart TD
    %% Visual Styles
    classDef user fill:#ffdfba,stroke:#ffb347,stroke-width:2px,color:#333
    classDef orch fill:#bae1ff,stroke:#5facff,stroke-width:2px,color:#333
    classDef agent fill:#baffc9,stroke:#42d669,stroke-width:2px,color:#333
    classDef doc fill:#ffffba,stroke:#e6e65a,stroke-width:2px,color:#333
    classDef cmd fill:#f3e8ff,stroke:#c084fc,stroke-width:2px,color:#333

    %% Top Level: User
    U((👤 User)):::user
    
    %% Commands
    CmdPlan[⚡ /teams:plan <br> <i>Start new feature</i>]:::cmd
    CmdRun[⚡ /teams:run <br> <i>Resume existing</i>]:::cmd

    %% Middle Level: Orchestration
    O[🧠 Orchestrator]:::orch
    P[📝 PLAN.md]:::doc

    %% Bottom Level: Execution
    subgraph Team[🔄 Task Execution Loop]
        direction LR
        B[👷 Builder]:::agent <--> |"Build ➔ Review ➔ Fix"| V[🔎 Validator]:::agent
    end

    %% Routing (Top to Bottom)
    U --> CmdPlan
    U --> CmdRun

    CmdPlan --> |"1a. Discuss & Approve"| O
    CmdRun --> |"1b. Load existing"| O

    O -. "2. Creates / Reads" .-> P
    O -- "3. Spawns Native Team" --> Team

    %% Final Report loops back
    Team -- "4. All Tasks Complete" --> O
    O -- "5. Final Report" --> U
```
