# /plan — Plan a development task

Start JSAgentFlow planning for a new task.

## Usage
```
/plan agregar autenticación con JWT
/plan implementar CRUD de productos
```

## What this does

1. Load the planner skill
2. Run codebase analysis
3. Run gap analysis — detect ambiguities, ask questions by category
4. Wait for user to answer all gaps
5. Decompose into phased subtasks with agent assignments
6. Generate impact report (files, deps, risks)
7. Ask for approval

After approval, run `/execute` to start.

## Invoke

Load the planner skill and begin with Phase 1: Codebase Analysis for the task provided.
