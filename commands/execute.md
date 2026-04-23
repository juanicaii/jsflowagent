# /execute — Execute an approved plan

Execute the current JSAgentFlow plan phase by phase.

## Prerequisites

A plan must exist at `.jsagentflow/current-plan.md` with status `approved` or `executing`. If not, tell user to run `/plan` first.

## Pre-execution checks

Before starting, validate the plan:

1. Read and parse `.jsagentflow/current-plan.md` — verify YAML frontmatter is valid
2. Verify `status` is `approved` (fresh) or `executing` (resuming)
3. Verify all `agent` values in subtasks are valid skills (exist in `.claude/skills/`)
4. Check if any file in `creates` already exists on disk — warn the user if so
5. If any check fails, report the issue and ask user how to proceed

## Process

1. Set `status: executing` in the plan frontmatter
2. Find the first subtask where `status` is NOT `completed` or `skipped`
3. For each remaining phase:
   - Display phase header with subtask list and progress so far
   - For each subtask in the phase: load the corresponding agent skill (`db-agent`, `backend-agent`, `ui-design`, `frontend-agent`, `qa-agent`, `devops-agent`, `security-agent`, `review-agent`) and invoke using the protocol below
   - If subtask FAILS: enter error recovery (see below)
   - If subtask passes: update it in frontmatter, continue
   - After all subtasks in a phase pass: increment `phases_completed`
4. After all phases complete: run the completion routine

## Subtask invocation protocol

When dispatching a subtask to an agent, construct this prompt:

```markdown
## Subtask {id}: {title}

### Objective
{objective}

### Allowed files
You may ONLY create or modify these files:

**Create:**
{creates — one per line, or "None"}

**Modify:**
{modifies — one per line, or "None"}

### Gap answers
{for each entry in gap_answers:}
- {question}: {answer}

### Previous phase outputs
{for each subtask ID in depends_on where status == "completed":}
#### Subtask {dep.id} — {dep.title} (COMPLETED)
{dep.output_summary}

### Design specs
{only if agent == "frontend-agent" AND a depends_on subtask has design_spec_path:}
Read the design spec file at: {design_spec_path}
You MUST follow these specs exactly — colors, spacing, typography, layout, states.
```

### After subtask completion

1. Parse the agent's structured output (the `## Subtask X.Y — ... — STATUS` block)
2. Update the subtask in plan frontmatter:
   - Set `status` to `completed`, `failed`, or `completed_with_bugs`
   - Set `output_summary` to the agent's full summary block
   - Clear `current_subtask`
3. If agent is `ui-design`: verify that the `design_spec_path` file was created
4. Write the updated frontmatter back to `.jsagentflow/current-plan.md`

### Dependency installation

Each agent is responsible for installing dependencies it needs using the project's package manager (detected during planning). Agents' `allowed-tools` already include `Bash(npm *)`, `Bash(pnpm *)`, `Bash(yarn *)`. All dependencies must have been listed in the impact report.

## Error recovery

When a subtask fails:

1. Update the subtask's `status` to `failed` in plan frontmatter
2. Save the error details to `output_summary`
3. Present the user with these options:

### a) Retry
Re-run the same subtask. Files created during the failed attempt remain on disk — the agent will overwrite them. The user may fix external issues (env vars, database, missing dependency) before retrying.

### b) Skip
Mark subtask as `skipped` and continue to the next. **Warning**: if the skipped subtask has dependents (other subtasks list it in `depends_on`), warn the user that those may also fail. Ask for confirmation before skipping.

### c) Fix and continue
The user manually fixes the issue in their editor, then says "continue". Mark the subtask as `completed` with `output_summary: "Manually fixed by user"` and move on.

### d) Abort
Stop execution entirely. The plan stays at `.jsagentflow/current-plan.md` with `status: executing` and partial progress recorded in each subtask's `status`. User can run `/execute` later to resume from the first incomplete subtask.

### File cleanup on failure

Do NOT delete files created during a failed subtask. They serve as debugging context. The agent's failure report lists "Files partially created" — these will be overwritten on retry or cleaned up manually by the user.

## Resume

When `/execute` is invoked and a plan exists with `status: executing`:

1. Read subtasks from frontmatter
2. Show the user what was completed so far (list of completed/skipped subtasks)
3. Find the first subtask where `status` is `pending` or `failed`
4. Ask user to confirm resuming from that subtask
5. Continue normal execution from that point

## Completion routine

When all phases finish successfully:

1. Display completion report (files created, modified, tests passed, agents used)
2. Move the plan to history:
   - Create `.jsagentflow/history/` if it doesn't exist
   - Move `.jsagentflow/current-plan.md` to `.jsagentflow/history/YYYY-MM-DD_{task_slug}.md`
   - Update the plan's frontmatter: set `status: completed` and add `completed: "ISO-8601 timestamp"`
3. Append to `.jsagentflow/changelog.md` (create if doesn't exist):

```markdown
## [YYYY-MM-DD] Task title

**Agents:** Backend, UI Design, Frontend, QA
**Phases:** N completed

### Files created
- `path/to/file.ts` — brief description

### Files modified
- `path/to/existing.ts` — what changed

### Dependencies added
- `package@version` — purpose

### Decisions made
- JWT library: jose (chosen over jsonwebtoken)
- Password hashing: bcrypt
- [other gap analysis answers]

---
```

4. If Memory MCP is available, also save a summary to memory with tags `jsagentflow`, `changelog`, and the `task_slug`
