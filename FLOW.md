# Default Execution Flow

This diagram illustrates the sequential flow for the `/teams:plan` skill, which relies on Native Agent Teams. The Orchestrator drives the high-level plan, while the Builder and Validator operate in a task-completion loop.

```mermaid
sequenceDiagram
    actor User
    participant Orch as Orchestrator
    participant Plan as .build/PLAN.md
    participant Team as Agent Team (Builder + Validator)

    %% Planning Phase
    User->>Orch: /teams:plan
    Orch->>Plan: Drafts Tasks & Criteria
    Orch-->>Plan: (Optional AI Review)
    Orch->>User: Request Approval
    User->>Orch: Approves

    %% Execution Phase
    Orch->>Team: Spawns Builder & Validator
    Orch->>Team: Adds Tasks to Shared List

    %% Internal Team loop
    loop For each task
        Team->>Team: Builder claims & implements task
        Team->>Team: Builder messages Validator for review
        alt If FAIL
            Team->>Team: Validator replies FAIL (max 2)
            Team->>Team: Builder fixes code
        else If PASS
            Team->>Team: Validator replies PASS
            Team->>Team: Builder marks task complete
        end
    end

    %% Completion
    Team->>Orch: All Tasks Completed
    Orch->>User: Final Success Report
```
