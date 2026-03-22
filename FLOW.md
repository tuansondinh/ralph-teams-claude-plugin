# Default Execution Flow

This diagram illustrates the default flow for the `/teams:plan` skill, which relies on Native Agent Teams. The Orchestrator drives the high-level plan, while the Builder and Validator operate in a parallel loop to complete tasks.

```mermaid
flowchart TD
    %% Roles
    User([User])
    Orch[Orchestrator]
    Team[Agent Team <br>(Builder + Validator)]
    Tasks[(Shared Task List)]

    %% Flow
    User -- "/teams:plan" --> Orch
    
    subgraph Plan Phase
        direction TB
        Orch -- "Drafts & Refines" --> Plan[(.build/PLAN.md)]
        Orch -. "Optional AI Review" .-> Plan
    end
    
    Plan -- "Approves" --> User
    User -- "Starts Build" --> Orch

    subgraph Execution Phase
        direction TB
        Orch -- "Spawns Team" --> Team
        Orch -- "Adds Tasks" --> Tasks
        Team <--> |"Claims & Updates Tasks"| Tasks
        
        %% Internal Team loop
        Team -. "Builder & Validator <br> communicate directly <br> until task passes" .-> Team
    end

    Tasks == "All Tasks Done" ===> Orch
    Orch -- "Final Report" --> User
```
