# Iterative specs: goal first, requirements with criteria, approaches, a definition of done

**Spec version:** 1
**Owner:** Will (`willdah`)
**Parent:** [create-it.md](create-it.md) v10

## Problem

plan-it starts interviewing before it confirms what problem the work solves, never states functional and non-functional requirements for the user to confirm, and writes the criteria and decisions as one fixed plan. That makes specs read like waterfall: the first solution the interview lands on gets built, alternatives go unseen, and nothing tells build-it which later changes to keep cheap. Users need plan-it to confirm the goal, confirm the requirements, compare approaches, and write a spec for the smallest version that solves the problem, and they need build-it and review-it to hold every change to one definition of done, so that code stays easy to maintain and extend.

## Requirements

### Functional

- **F1:** plan-it confirms the goal before it interviews.
  - [ ] F1.1: When the invocation or the conversation states a goal, plan-it restates it in one or two sentences and asks the user to confirm it before the first interview question.
  - [ ] F1.2: When no goal is stated, plan-it asks what the goal is, what problem it solves and for whom, before any other question.
- **F2:** plan-it confirms the requirements before it compares approaches.
  - [ ] F2.1: After the interview, plan-it shows the functional and non-functional requirements as two lists and waits for the user's yes or edits.
  - [ ] F2.2: Interview questions before this step are about the problem and its constraints, not about a solution.
- **F3:** plan-it proposes approaches before it writes the spec.
  - [ ] F3.1: After the requirements are confirmed, plan-it presents up to 3 distinct approaches, each with how it meets the requirements, its cost (effort, new dependencies, risk) and its effect on the Future follow-ups, and marks one as recommended with the reason.
  - [ ] F3.2: When only one approach is sensible, plan-it says so and why instead of inventing options.
  - [ ] F3.3: After the user picks an approach, plan-it interviews only the decisions that approach leaves open and that are costly to reverse.
- **F4:** The spec separates what is built now from what comes later.
  - [ ] F4.1: The spec template has, in order: Problem, Requirements (Functional, Non-functional), Out of scope, Future follow-ups, Testing (optional), Decisions, Changelog. It has no separate Acceptance criteria section.
  - [ ] F4.2: Each requirement has an ID (`F1`, `N1`) and one or more nested checkbox criteria (`F1.1`), in EARS form for behaviour, each checkable from the diff, a command or a screenshot. IDs are never renumbered or reused across versions; a dropped criterion keeps its ID, struck through, with the version that dropped it.
  - [ ] F4.3: Non-functional requirements are specific to the work and have a concrete bound where one exists; generic quality goes in the definition of done, not the spec.
  - [ ] F4.4: Out of scope lists plausible goals deliberately not taken, each with a reason. Future follow-ups lists likely next iterations, one line each, that are not built now.
  - [ ] F4.5: Decisions record only choices that are costly to reverse or that the user has a stake in, each with what was rejected and when to revisit it. The spec has no file layout and no build order.
- **F5:** Large work splits into vertical slices.
  - [ ] F5.1: A spec fits one reviewable PR, about 2–5 requirements and 3–10 criteria; more is the signal to split.
  - [ ] F5.2: Each child spec delivers a thin end-to-end piece the user can see working, never one layer; the first child is the smallest piece that proves the approach. The approach is chosen once, in the parent.
- **F6:** build-it and review-it apply one definition of done to every change.
  - [ ] F6.1: build-it and review-it each contain the same definition of done: criteria met with evidence; gates green and the failing-test rule kept; each concern in one place, domain names, no dead code; nothing built for a Future follow-up and nothing that blocks one; no option, layer or abstraction that no requirement needs.
  - [ ] F6.2: review-it reports a red gate or a missing required test against the definition of done as Must fix, and a blocked follow-up or an abstraction no requirement needs as Should fix.
  - [ ] F6.3: review-it reports a requirement with no criteria as Must fix.
- **F7:** The other skills read the new spec shape and still read the old one.
  - [ ] F7.1: build-it, stage-it and review-it take the criteria from the checkboxes under `## Requirements`, cite them by ID (`F1.1`), and treat a spec with a flat `## Acceptance criteria` section (`AC1`…) the same way.
  - [ ] F7.2: With no spec, build-it, stage-it and review-it write 1–3 requirements, each with its criteria, and continue only when the user confirms.
  - [ ] F7.3: The create-it driver says the user takes part only in plan-it, not only in the interview.

### Non-functional

- **N1 Agent-neutral:** each skill still works when loaded alone.
  - [ ] N1.1: No skill reads a file outside its own folder; the definition of done is copied, not referenced.
- **N2 Not overridable:** config cannot skip the new steps.
  - [ ] N2.1: plan-it's "Config cannot change" line names the goal check, the requirements check and the approaches step.

## Out of scope

- A drift check between the two copies of the definition of done: the user chose not to have one.
- Migrating existing specs to the new shape: old flat-criteria specs keep working (F7.1).
- Plan or task documents in the style of Spec Kit or Kiro: build-it owns how the work is done.
- An eval suite or automated tests for the skills (as in the parent spec).

## Future follow-ups

- explore-it proposes repo-specific definition-of-done items under `## Rules` from confirmed patterns.
- A "do nothing" or non-code option as a standing prompt in the approaches step.

## Testing

No automated tests (parent spec, Out of scope). Each criterion is checked by reading the changed skill files; `claude plugin validate` runs as the repo gate.

## Decisions

- **Goal, requirements and approaches are plan-it's own steps, not part of the interview.** An interview that runs before the goal is fixed explores the wrong problem.
  Rejected: one confirmation at the end of the interview (fewer turns, but the interview can drift first).
- **Requirements carry their criteria; no separate Acceptance criteria section.** Matches Kiro's nested criteria and agile practice, and the user confirms exactly what gates the PR.
  Rejected: separate Requirements and Acceptance criteria lists (two lists that drift); flat `AC1` IDs tagged with a requirement (two numbering systems).
- **The name "acceptance criteria" stays** for the nested checkboxes; it is the industry term and keeps downstream text stable.
  Rejected: renaming them "checks" (about 30 edits, off-standard).
- **Maintainability is a definition of done, not a per-spec requirement.** It applies to every change, as agile definitions of done do.
  Rejected: an N-requirement in each spec (repeated, and easy to drop).
- **The definition of done is copied into build-it and review-it, with no drift check.** Keeps each skill self-contained, the house pattern.
  Rejected: one shared file both skills read (needs a path outside the skill folder; not guaranteed on every agent).
- **Approaches come after requirements and before the spec.** Approaches need requirements to be judged against; the spec records the chosen one, and the others go under Decisions as rejected.
- **Vertical slices; approach chosen in the parent.** Layer slices ship nothing the user can see.
- **Old flat-criteria specs keep working.** Revisit when no open spec uses `AC` IDs.
- **This spec ships as one PR despite exceeding F5.1's size.** Every change is markdown in one plugin, so one PR stays reviewable.
  Rejected: three slices (plan-it flow; spec shape and readers; definition of done).

## Changelog

- v1 (2026-11-26): first version.
