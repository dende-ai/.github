---
name: dende-pr
description: "Open or update a pull request in a Dende repository: the fresh-context review before the PR, the PR body, the evidence table and the handling of review findings. Use when the change is ready to push."
---

# Before the PR

1. Run the repository's check command. Push nothing that fails it.
2. Review the diff in a fresh context that cannot edit: a read-only subagent or review agent.
   Give it the diff, the acceptance examples, `CODE_STYLE.md` and the repository's `AGENTS.md`, and ask:
   "Which acceptance example has no test? Which test does not prove its example? What changed outside the issue?
   Which code breaks `CODE_STYLE.md`?"
3. Fix what it finds, or write in the PR why not.

# PR body

Fill in the repository's PR template. Every section stays; write `none` where nothing applies.

```markdown
Your review: code

## Issue and owner

Linear issue and human assignee: TECH-<n> · <owner>
Prepared by: <agent and harness> · protocol <revision from AGENT_PROTOCOL.md>

## Goal

<one or two sentences from the issue>

## Changes

<one line per file or behavior>

## Checks run

Revision `<sha>`

| Check | Result |
| --- | --- |
| <check command> | passed · failed · not run |

## Risks

<what can break, how to undo>
Design: <Figma link> · none
Tracking: <event> · none
Alarm: <alarm and threshold> · none
Linked: <repository: issue> · none
Evidence: <environment where the acceptance examples ran>
Proof: <link to a screenshot of each changed screen state, or to a chart or an HTML page of the measured numbers> · none

## Side findings

<one line each; no work on them without a decision>
```

- `AGENTS.md` says whether the first line is `Your review: code` or `Your review: timing only`.
- Write the Proof line when the change has a visible result or a measured number. Attach the files to the issue,
  not to the repository, and link them.
- Name only this PR's issue. Write `Part of TECH-<n>` while more PRs follow, `Closes TECH-<n>` on the last one.
- Assign the owner.

# Review findings

- Codex posts a review summary on the PR. Without it the PR is not reviewed.
- Fix each P0 and P1 finding, or reply on its thread with the reason. Push, then ask for the review again.
- A finding that comes up on two PRs is a rule candidate: open an issue in dende-agent-protocol with both cases.

# Merge

- The owner gives the OK in the issue, except for the standing-OK classes in `AGENT_PROTOCOL.md`.
- After the merge, write in the issue the merged revision and where the change runs. The issue closes when the
  acceptance examples pass there.
