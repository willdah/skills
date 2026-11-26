# create-it as an orchestrator: each phase in its own session

**Spec version:** 1
**Owner:** Will (`willdah`)
**Parent:** [create-it.md](create-it.md) v10

## Problem

`/create-it` runs every phase in one session with the Skill tool. By the review rounds, the context holds the author's reasoning, diffs, test output and earlier findings. That biases review-it toward the author's view and lets stale detail crowd out what matters on long runs. create-it users need the driver to stay a thin orchestrator that keeps only each phase's result, while every phase after planning runs in its own isolated session, and every review round starts with no memory of the build or of earlier reviews.

## Requirements

### Functional

- **F1 Orchestrator:** `/create-it` plans inline with the user, then delegates every later phase to a new isolated session.
  - [ ] F1.1: The create-it skill shall run plan-it inline in its own session, with the Skill tool, as it does today.
  - [ ] F1.2: When plan-it has stored the spec, create-it shall run build-it, stage-it, each review-it round and each build-it fix round, each in a new agent session (in Claude Code, the `Agent` tool), and shall never continue an earlier session.
  - [ ] F1.3: The prompt for each delegated session shall tell it to run the phase skill with arguments that start `called by create-it:`, and shall pass only what the phase needs: the spec reference, the branch or PR, the round number, and for stage-it the assumptions list.
- **F2 Report schema:** every phase called by create-it ends with one report in a shared schema, and create-it decides only from those reports.
  - [ ] F2.1: The create-it skill shall define the report once: a fenced block tagged `create-it-report` of flat `key: value` lines, lists as `- ` items, with the keys `phase`, `status` (`done` or `blocked`), `blocked`, `spec`, `branch`, `pr`, `criteria`, `assumptions`, `verdict`, `must_fix`, `spec_stale`, `ready`. A phase fills the keys it owns and writes `none` for the rest.
  - [ ] F2.2: create-it shall paste that schema into every delegated prompt.
  - [ ] F2.3: The "Called by create-it" paragraph of plan-it, build-it, stage-it and review-it shall tell the phase to end with the `create-it-report` block, and name the keys it fills.
  - [ ] F2.4: create-it shall make every loop and stop decision (red gate, stale spec, open Must fix, the same Must fix twice, the round limit) from the report fields only.
  - [ ] F2.5: If a delegated session returns no `create-it-report` block, or a block create-it cannot read, then create-it shall treat that phase as `blocked` and go to the summary.
- **F3 Unbiased review:** each review round is independent of the build and of earlier rounds.
  - [ ] F3.1: The prompt for a review-it session shall contain the PR, the spec reference and the round number, and shall not contain build assumptions, earlier findings or any build reasoning.
  - [ ] F3.2: When review-it is called by create-it, it shall do step 2 in its own session and not start a nested subagent.
- **F4 Fallback:** create-it still runs on an agent with no subagents.
  - [ ] F4.1: If the agent cannot start a subagent, then create-it shall run the phases inline with the Skill tool, say `no subagents; running phases inline (review not isolated)` once, and repeat that warning in the summary.
- **F5 Docs:** the plugin docs match the new behaviour.
  - [ ] F5.1: The README describes `/create-it` as an orchestrator that runs each phase after planning, and each review round, in a new session.
  - [ ] F5.2: `create-it.md` AC12 carries an amendment note that points to this spec, and its Spec version and Changelog are bumped.
  - [ ] F5.3: `plugin.json` is at version `0.5.0`.

### Non-functional

- **N1 Context:** the orchestrator's context holds no phase working detail (diffs, test output, review reasoning), only the reports. Judgment call.
  - [ ] N1.1: The create-it skill text tells the orchestrator to keep only the report fields, and not to read diffs, logs or findings itself.
- **N2 Portability:** the skill text stays agent-neutral.
  - [ ] N2.1: Claude Code tool names (`Agent`, `Skill`) appear in the create-it skill only as examples, next to a neutral term ("a new agent session", "the skill tool").
- **N3 Compatibility:** phase skills called directly by the user behave as before.
  - [ ] N3.1: The diff changes only the "Called by create-it" paragraphs of the phase skills, and no other step.

## Out of scope

- Running plan-it in its own session with the orchestrator relaying the interview: the user chose inline, because a relay costs one round trip per question.
- Messages from a phase to the orchestrator during the run: the user chose one report at the end, to keep the orchestrator's context small and stay agent-neutral.
- Continuing the first build session for fix rounds: each fix round reads the findings fresh from the PR.
- Refusing to run without subagents: the fallback with a warning is enough.

## Future follow-ups

- Ship Claude Code subagent definitions per phase (`agents/review-it.md` with no edit tools), so "review never edits" is enforced.
- Pick a model or effort per phase session.
- Run independent phases or child specs in parallel sessions.
- Relay plan-it's interview from its own session.

## Testing

No automated tests (parent spec, Out of scope). Each criterion is checked by reading the changed skill files; `claude plugin validate` runs as the repo gate.

## Decisions

- **Plan inline, delegate the rest.** The interview is the context the orchestrator needs, and the session is still clean when it runs. Rejected: plan-it in its own session with a relay, because one round trip per question makes the interview clumsy. Revisit if a long interview crowds the orchestrator.
- **One report at the end, in a fenced `key: value` block.** Flat text works on any agent and models write it reliably. Rejected: messages during the run (pulls phase detail back in, Claude Code only); JSON (models break it more often inside prose).
- **The schema lives in create-it and goes into every prompt.** One source of truth, and phases called directly are unchanged. Rejected: a shared `docs/report.md` read by each phase, because a path relative to an installed skill folder breaks across agents and install layouts; per-phase Claude Code agent definitions, because they are Claude Code only (kept as a follow-up).
- **A new session per review round and per fix round.** A reviewer that never saw the build or earlier rounds reads the change as a colleague would; a fixer reads the findings as written instead of defending its own choices.
- **Fall back to inline, with a warning.** Rejected: refusing to run, because the run still produces a useful PR and the warning tells the user the review was not independent.

## Changelog

- v1 (2026-11-26): first version.
