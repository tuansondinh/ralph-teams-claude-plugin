# Default Execution Flow

This diagram provides a high-level, abstract overview of how the `/teams:plan` skill operates. The Orchestrator handles planning and user interaction, while the Builder and Validator communicate directly to complete tasks.

```mermaid
flowchart TD
    %% Visual Styles
    classDef user fill:#ffdfba,stroke:#ffb347,stroke-width:2px,color:#333
    classDef orch fill:#bae1ff,stroke:#5facff,stroke-width:2px,color:#333
    classDef agent fill:#baffc9,stroke:#42d669,stroke-width:2px,color:#333
    classDef doc fill:#ffffba,stroke:#e6e65a,stroke-width:2px,color:#333

    %% Nodes
    U((👤 User)):::user
    O[🧠 Orchestrator]:::orch
    P[📝 PLAN.md]:::doc
    B[👷 Builder]:::agent
    V[🔎 Validator]:::agent

    %% Connections
    U -- "1. Discuss & Approve" --> O
    O -. "2. Creates" .-> P
    O -- "3. Spawns Native Team" --> B & V

    subgraph Team[🔄 Task Execution Loop]
        direction LR
        B <--> |"4. Build ➔ Review ➔ Fix"| V
    end

    Team -- "5. All Tasks Complete" --> O
    O -- "6. Final Report" --> U
```
