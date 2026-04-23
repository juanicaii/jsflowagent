# JSFlow

Multi-agent orchestration plugin for Claude Code. Decomposes complex development tasks into phased subtasks executed by specialized agents (Backend, UI Design, Frontend, QA).

## Installation

```bash
# From GitHub
claude plugin install jsflow@tu-usuario/jsflowagent

# From local path (for development)
claude --plugin-dir /path/to/jsflowagent
```

## Quick Start

```bash
/jsflow:plan "add JWT authentication"    # Plan the task
# Review impact report → approve
/jsflow:execute                           # Execute phase by phase
/jsflow:review-plan                       # Check progress anytime
```

## Structure

```
jsflowagent/
├── .claude-plugin/
│   └── plugin.json                    # Plugin manifest (name: jsflow)
├── skills/
│   ├── planner/SKILL.md              # Task decomposition & gap analysis
│   ├── backend-agent/SKILL.md        # Backend implementation agent
│   ├── ui-design/SKILL.md            # UI/UX design agent
│   ├── frontend-agent/SKILL.md       # Frontend implementation agent
│   └── qa-agent/SKILL.md             # Testing & QA agent
├── commands/
│   ├── plan.md                        # /jsflow:plan command
│   ├── execute.md                     # /jsflow:execute command
│   └── review-plan.md                # /jsflow:review-plan command
├── RULES.md                           # System rules for all agents
└── .gitignore
```

## Commands

| Command | Description |
|---------|-------------|
| `/jsflow:plan <task>` | Analyze codebase, detect gaps, generate execution plan |
| `/jsflow:execute` | Run the approved plan phase by phase |
| `/jsflow:review-plan` | View current plan status and remaining work |

## Pipeline

```
/jsflow:plan → Backend Agent → UI Design → Frontend Agent → QA Agent
                    ↓              ↓             ↓              ↓
              endpoints,      component     implement        tests,
              schemas,        specs,        designs,         bug
              migrations      tokens        components       reports
```

## How it works

1. **Planner** scans your codebase, identifies gaps/ambiguities, and generates a phased execution plan with impact report
2. **You** review and approve the plan
3. **Sub-agents** execute each phase sequentially:
   - **Backend** creates endpoints, schemas, and business logic
   - **UI Design** produces component specs, visual direction, and design tokens
   - **Frontend** implements the designs with full accessibility
   - **QA** writes and runs tests, reports bugs
4. **State** is tracked in `.jsagentflow/current-plan.md` during execution
5. **History** is saved to `.jsagentflow/history/` and `.jsagentflow/changelog.md`

## License

MIT
