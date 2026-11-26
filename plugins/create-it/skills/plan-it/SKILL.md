---
name: plan-it
description: "Plan a piece of work: confirm the goal, interview the user, confirm functional and non-functional requirements, propose up to 3 approaches, then write a versioned spec (problem, requirements with nested EARS acceptance criteria and stable IDs, out of scope, future follow-ups, testing, decisions) that build-it, stage-it and review-it check against. Use when the user wants to plan, scope, spec or pin down requirements for a feature, fix or change, even if they only describe the idea."
metadata:
  credits:
    inspired_by:
      - skill: grilling
        author: Matt Pocock
        source: https://github.com/mattpocock/skills
        took: one question at a time, a recommended answer with each, explore the code instead of asking, walk every branch
      - skill: to-spec
        author: Matt Pocock
        source: https://github.com/mattpocock/skills
        took: read the repo's domain docs first and use their terms; confirm the testing seam
      - skill: to-tickets
        author: Matt Pocock
        source: https://github.com/mattpocock/skills
        took: slices that ship on their own, blocking edges, user approves the breakdown
      - skill: caveman
        author: Julius Brussee
        source: https://github.com/JuliusBrussee/caveman
        took: terse tone for agent chat
---

Turn what the user wants into a spec that the other create-it skills check work against. The order is fixed: goal, interview, requirements, approaches, spec. A spec written before the goal and requirements are settled sends build-it after the wrong target, and every later step inherits the error.

A spec describes the smallest version that solves the confirmed problem, not a full plan. It says what must be true and why; build-it decides how. Code keeps changing after this spec ships, so the spec also names the likely next steps, so that build-it keeps them cheap without building them.

**Called by create-it** (arguments start `called by create-it:`): do this phase, chain to nothing (skip `## Then`), and end with the `create-it-report` block the caller gives you, filling `phase`, `status`, `blocked` and `spec`; when you split the work, `spec` is the first child that nothing blocks, and list the other children's references above the block.

## Tone

Agent chat (status, progress, hand-off): terse, fragments OK. The spec: short full sentences, no filler or hedging. Interview questions, approaches and anything the user asks to clarify: normal prose.

## 0. Load config

Read if present: `docs/create-it/plan.md` (repo), then `~/.config/create-it/plan.md` (user). Also read `## Code Host` from `docs/create-it/stage.md`, else from `~/.config/create-it/stage.md`.

- `## Then`, `## Code Host`: the repo value wins, else the user value, else the defaults.
- `## Rules`, `## Gates`: apply the items from both files. A user file cannot drop or weaken a repo item. Gates here are checks the spec must pass before hand-off.
- Any other text is context.

Say in one line what you loaded, for example `config: repo docs/create-it/plan.md, user none`. With no files: `no create-it config; using defaults (run /explore-it to tailor)`.

**Config cannot change:** you never merge a PR; you confirm the goal (step 2), interview before the spec, confirm the requirements (step 4) and propose approaches (step 5) before writing the spec.

## 1. Read the domain

Before the first question, find and read the docs that record how this repo thinks: a glossary or `CONTEXT.md`, ADRs (`docs/adr/` or similar), architecture or design docs, `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, the README, `ENV_MAP.md`. Names vary; look for what exists. Use the repo's terms in the questions and the spec, and stay inside its recorded decisions unless the user chooses to change one. This saves the user from explaining what is already written down.

## 2. Confirm the goal

- The request or the conversation states a goal: restate it in one or two sentences (the problem, and for whom) and ask the user to confirm or correct it.
- No goal stated: ask what the goal is, what problem it solves, and for whom.

Ask nothing else until the user confirms the goal. An interview that starts before the goal is fixed explores the wrong problem.

## 3. Interview

- Ask one question at a time. Several at once are hard to answer well.
- Give your recommended answer with each question, so the user can accept in a word.
- When the code can answer a question, read the code instead of asking.
- Ask about the problem: who uses it, what must happen, the constraints (load, security, compatibility, deadlines). Do not pick a solution yet; approaches come in step 5.
- Push optional extras to Future follow-ups instead of requirements, so the spec stays the smallest version that solves the problem.
- Resolve the questions that others depend on first.

## 4. Confirm the requirements

Show the requirements as two short lists and wait for the user's yes or edits:

- **Functional:** what the system must do, from the user's or caller's point of view.
- **Non-functional:** qualities specific to this work (performance, security, accessibility, compatibility, operability), each with a concrete bound where one exists (`p95 < 200ms`, not "fast"). Mark one that cannot be bounded as a judgment call. Generic quality (tested, maintainable, easy to extend) is not a requirement here; build-it and review-it apply it to every change.

## 5. Propose approaches

Present up to 3 distinct ways to meet the confirmed requirements. For each, one short paragraph and:

- how well it meets the requirements;
- its cost: effort, new dependencies, risk;
- its effect on the likely follow-ups: which it keeps cheap, which it makes harder.

Mark one as recommended and say why. When only one approach is sensible, say so and why; do not invent options to fill the list. Mention doing nothing or a non-code fix only when it is a real option. The user picks one or asks to combine them.

Then interview only the decisions the chosen approach leaves open that are costly to reverse: interfaces that other code or users depend on, data that is stored, new dependencies, trade-offs the user cares about. Confirm the testing seam: the highest existing public interface where the change can be checked. Prefer one seam. Internal structure, file layout and build order belong to build-it.

Write the spec only after the user confirms the approach and these decisions.

## 6. Write the spec

```markdown
# <title>

**Spec version:** 1

## Problem
One paragraph: what is wrong or missing, and for whom.

## Requirements
### Functional
- **F1:** <what the system must do>.
  - [ ] F1.1: When <trigger>, the <system> shall <response>.
  - [ ] F1.2: <checkable statement>.
### Non-functional
- **N1 <quality>:** <bound specific to this work>.
  - [ ] N1.1: <how it is checked, for example a benchmark or test>.

## Out of scope
- <A plausible goal we deliberately do not take>: <why>.

## Future follow-ups
- <A likely next iteration, one line. Not built now.>

## Testing
Where the change is checked (the seam) and how.

## Decisions
- **<Decision>.** <Reason and trade-off.>
  Rejected: <the alternative and why>. <When to revisit, if it applies.>

## Changelog
- v1 (<date>): first version.
```

Requirement rules, because build-it, the PR checklist and review-it all cite them:

- Each requirement has an ID (`F1`, `N1`) and one or more acceptance criteria nested under it (`F1.1`). The acceptance criteria are those checkboxes; there is no separate criteria section.
- Each criterion is checkable from the diff, a command or a screenshot. A requirement with no criterion is not done.
- Behaviour criteria use EARS: `When <trigger>, the <system> shall <response>` (or `While <state>, …`, `If <unwanted condition>, then …`, or `The <system> shall …` for always-true behaviour). Other criteria are plain checkable statements.
- IDs are stable. Never renumber or reuse a requirement or criterion ID across versions. A dropped one keeps its ID, struck through, with the version that dropped it.
- No user stories. They add length that cannot be checked against a diff.

Section rules:

- **Out of scope** holds plausible goals deliberately not taken, each with a reason; not negated goals like "it should not crash".
- **Future follow-ups** holds likely next iterations, one line each. They are not commitments. build-it keeps them cheap and builds none of them.
- **Decisions** hold only choices that are costly to reverse or that the user has a stake in, each with what was rejected (the other approaches from step 5 among them) and when to revisit. No file layout, class names or build order.

Leave out **Testing** when the work has nothing to test. Add other sections (constraints, open questions, links) when the interview produced them. If the tracker has a house issue template for specs or features, fit the spec into it and keep the core sections. Apply every `## Rules` item and pass every `## Gates` item.

## 7. Split large work

One spec fits one reviewable PR: about 2–5 requirements and 3–10 criteria. More is the signal to split into child specs.

- Split into vertical slices: each child delivers a thin piece the user can see working, end to end. Never split by layer (database, then API, then UI); a layer alone ships nothing.
- The first child is the smallest piece that proves the approach.
- The parent holds the problem, the requirements and the chosen approach for the whole effort, and lists its children in dependency order. Children do not propose approaches again unless a slice finds something new.
- Each child is a full spec with its own IDs, links its parent, and has a `Blocked by:` line (`none` when it can start at once).
- Children are committed work that the problem needs. Future follow-ups are not.

Show the breakdown first: per child, its title, what it delivers and what blocks it. Store nothing until the user approves it.

## 8. Store the spec

Where config says. Default:

- The repo has a code host or tracker: create an issue whose body is the spec (`gh issue create`, or the `## Code Host` commands). Report the reference.
- No code host or tracker: write `docs/specs/<slug>.md` and do not commit it; build-it commits it first on its branch.

**Changing an existing spec:** bump `Spec version`, add a changelog line saying what changed and why, and update it in place. Only plan-it edits specs.

## 9. Hand off

Give the user the spec reference (`#N`, URL or path). If `## Then` is set, call that skill with the reference.
