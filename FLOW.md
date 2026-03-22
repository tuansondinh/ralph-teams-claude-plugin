# Default Execution Flow

This diagram illustrates the default flow for the `/teams:plan` skill, utilizing Native Agent Teams, the shared task list, and direct teammate communication.

```mermaid
flowchart TD
    %% Entities
    User([User])
    Orchestrator[Orchestrator / Team Lead]
    Reviewer[AI Reviewer <br>(Codex/Opus)]
    PlanFile[(.build/PLAN.md)]
    TaskList[(Shared Task List)]
    Builder[Builder Agent]
    Validator[Validator Agent]

    %% Planning Phase
    User -- "1. /teams:plan" --> Orchestrator
    Orchestrator -- "2. Discuss & Draft Plan" --> PlanFile
    Orchestrator -- "3. Ask for E2E Setup & Plan Review" --> User
    
    User -- "4. Approve Review (Yes)" --> Reviewer
    Reviewer -- "5. Review for missing edge cases/tasks" --> Orchestrator
    Orchestrator -- "6. Adjusts Plan" --> PlanFile
    
    Orchestrator -- "7. Show final plan & Ask approval" --> User

    %% Execution Phase
    User -- "8. Reply 'yes'" --> Orchestrator
    Orchestrator -- "9. Create Native Team" --> Builder & Validator
    Orchestrator -- "10. Add Tasks to List" --> TaskList

    %% Native Agent Team Execution Loop
    subgraph Native Agent Team
        direction TB
        Builder -- "11. Claim Task & Implement" --> TaskList
        Builder -- "12. Direct Message: Request Review (Commit SHA)" --> Validator
        Validator -- "13a. Direct Message: FAIL <br>(Max 2 Retries)" --> Builder
        Validator -- "13b. Direct Message: PASS" --> Builder
        Builder -- "14. Mark Complete & Claim Next Task" --> TaskList
    end

    %% Orchestrator Monitoring
    TaskList -. "15. Monitor Task Progress" .-> Orchestrator
    Orchestrator -. "16. Print Progress to User<br>(Building/Validating/Retrying...)" .-> User

    %% Completion
    TaskList == "All Tasks Complete" ===> Orchestrator
    Orchestrator -- "17. Shutdown Team & Cleanup" --> Builder & Validator
    Orchestrator -- "18. Final Success Report" --> User
```
