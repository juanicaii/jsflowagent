# JSFlow — AI Agent Orchestrator

## What is this

This plugin provides a multi-agent orchestration system built on Claude Code skills and sub-agents. Instead of doing everything in a single session, complex development tasks are decomposed by a Planner agent, validated by the user, and executed by specialized sub-agents (Backend, UI Design, Frontend, QA).

## Core workflow

1. User invokes `/jsflow:plan "task description"` to start
2. Planner skill activates: scans codebase, runs gap analysis, generates execution plan
3. User reviews the impact report and approves, edits, or rejects
4. User invokes `/jsflow:execute` to run the approved plan
5. Sub-agents execute each phase sequentially, reporting results after each step
6. User reviews final completion report

## Rules for all agents

1. NEVER modify files not listed in the current subtask
2. ALWAYS use snake_case for database fields, camelCase for code variables
3. ALWAYS follow existing project patterns detected during codebase analysis
4. NEVER install dependencies without listing them in the impact report first
5. NEVER skip the approval gate — the user MUST approve before any file changes
6. When a subtask fails, STOP and report the error — do not continue to the next phase

## State management

The execution plan is stored as a markdown file at `.jsagentflow/current-plan.md` during active sessions. This file is created by the Planner after approval, updated after each phase completes, and moved to `.jsagentflow/history/` after successful completion.

## Conventions

- Plans are written in structured markdown with YAML frontmatter for machine-readable sections
- Impact reports use tables for file listings and risk assessments
- Sub-agents output a summary block at the end of each subtask
- All timestamps use ISO 8601 format
