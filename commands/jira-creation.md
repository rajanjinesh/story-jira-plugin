---
description: Orchestrate prd.md → story candidates → stories.md → approval → Jira Stories.
---

You are the orchestrator. You only coordinate: call subagents, pass outputs along, and handle the approval checkpoint. Never read the PRD, write stories, or touch Jira yourself.

## Pre-check
Before starting, confirm these subagents from this plugin are available: `story-jira:prd-reader`, `story-jira:story-writer`, `story-jira:jira-publisher`. Also confirm `prd.md` and `CLAUDE.md` exist in the project root. If anything is missing, stop, tell me what, and go to the Run summary.

## Flow
1. **prd-reader** — Ask it to read `prd.md` and return story candidates (title + one-line scope). Wait for it to finish.
2. **story-writer** — Only after step 1 succeeds. Pass it the full candidate list from step 1. It writes the stories to `stories.md` using the `story-creation` skill.
3. **Approval checkpoint** — Show a short summary of the stories in `stories.md` (title + one line each) and ask me to choose **Approve** or **Reject** (use AskUserQuestion). Do not continue without my explicit choice.
   - Approve → go to step 4.
   - Reject → stop, ask me what needs to change, and send nothing to Jira.
4. **jira-publisher** — Only after Approve. Tell it that I approved `stories.md`, and ask it to create one Jira Story per entry in `stories.md` and return the created issue keys.

If any step fails, stop there and go to the Run summary.

## Run summary
End every run, including early stops, with only this summary. Give a one-line reason for anything not successful.

- Orchestrator: `prd.md` found / not found; agents found (names) / missing (names); handoffs with counts (e.g. "7 story candidates passed to story-writer", "7 stories passed to jira-publisher"); flag any count mismatch between steps (e.g. "7 candidates from prd-reader, but only 6 stories saved")
- Subagent 1 (prd-reader): Successful (N story candidates) / Not successful (reason) / Not run (reason)
- Subagent 2 (story-writer): Successful (N stories saved to `stories.md`) / Not successful (reason) / Not run (reason)
- Approval: Approved / Rejected (my feedback) / Not reached (reason)
- Subagent 3 (jira-publisher): Successful (N Jira stories created: issue keys) / Not successful (reason) / Not run (reason)
