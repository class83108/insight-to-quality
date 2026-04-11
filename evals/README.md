# insight-to-quality Evals

This directory contains evaluation scenarios for the `insight-to-quality` skill suite. Each scenario is a small, real-ish problem that exercises the full discovery → implementation workflow.

## Purpose

Two goals, run together:

1. **Smoke-test the workflow** — walk through every skill in sequence and confirm the handoffs work end-to-end
2. **Assess skill quality** — after each skill run, score the output against the rubric to find gaps in prompting, blocking logic, or output shape

## Directory Structure

```
evals/
  README.md                       # This file
  {scenario-name}/
    scenario.md                   # Problem brief — hand this to Claude to start the eval
    rubric.md                     # Per-skill scoring criteria
    runs/
      YYYY-MM-DD/                 # One directory per eval run
        00-start-prompt.md        # Exact prompt used to kick off the run
        01-goals-discovery.md     # Skill output (copy from conversation)
        02-dominant-ops.md
        03-system-map.md
        04-align-internals.md
        05-align-surface.md
        06-feature-planning.md
        07-feature-coverage.md
        08-gherkin.md
        09-tdd-workflow.md
        10-design-review.md
        11-pre-complete.md
        scores.md                 # Rubric scores + commentary for this run
```

## How to Run an Eval

1. Start a **fresh Claude Code session** in a temporary directory (or a git worktree)
2. Install the plugin: tell Claude which CLAUDE.md rules apply
3. Open `scenario.md` and paste the **User Context** block as your first message — do not add information not in that block
4. Trigger each skill in sequence using the slash commands
5. After each skill, copy the full output into the corresponding `runs/YYYY-MM-DD/NN-skill-name.md` file
6. After all skills are done, score each output against `rubric.md` and fill in `runs/YYYY-MM-DD/scores.md`

## Scoring Scale

Each skill is scored on three dimensions:

| Score | Meaning |
|-------|---------|
| **Pass** | Output meets all required criteria; next skill can proceed without manual correction |
| **Partial** | Output is usable but has a gap that required manual intervention or will create downstream noise |
| **Fail** | Output is missing something critical, or the skill did not block when it should have |

A **Partial** on a blocking check (e.g., feature-planning not blocking when SYSTEM_MAP is missing) is treated as **Fail**.

## Current Scenarios

| Scenario | Skills Covered | Status |
|----------|---------------|--------|
| [expense-tracker](expense-tracker/) | Full workflow (all 11 skills) | Ready |
