---
name: plan-it
description: "Plan a piece of work: interview the user until every decision is resolved, then write a versioned spec (problem, EARS acceptance criteria with stable IDs, out of scope, decisions, testing) that build-it, stage-it and review-it check against. Use when the user wants to plan, scope, spec or pin down requirements for a feature, fix or change, even if they only describe the idea."
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

Turn what the user wants into a spec that the other create-it skills check work against. The order is fixed: interview, then spec. A spec written before the decisions are settled sends build-it after the wrong target, and every later step inherits the error.

**Called by create-it** (arguments start `called by create-it:`): do this phase, chain to nothing (skip `## Then`), and return the spec reference.

## Tone

Agent chat (status, progress, hand-off): terse, fragments OK. The spec: short full sentences, no filler or hedging. Interview questions and anything the user asks to clarify: normal prose.

## 0. Load config

Read if present: `docs/create-it/plan.md` (repo), then `~/.config/create-it/plan.md` (user). Also read `## Code Host` from `docs/create-it/stage.md`, else from `~/.config/create-it/stage.md`.

- `## Then`, `## Code Host`: the repo value wins, else the user value, else the defaults.
- `## Rules`, `## Gates`: apply the items from both files. A user file cannot drop or weaken a repo item. Gates here are checks the spec must pass before hand-off.
- Any other text is context.

Say in one line what you loaded, for example `config: repo docs/create-it/plan.md, user none`. With no files: `no create-it config; using defaults (run /explore-it to tailor)`.

**Config cannot change:** you never merge a PR; you always interview before writing the spec.

## 1. Read the domain

Before the first question, find and read the docs that record how this repo thinks: a glossary or `CONTEXT.md`, ADRs (`docs/adr/` or similar), architecture or design docs, `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, the README, `ENV_MAP.md`. Names vary; look for what exists. Use the repo's terms in the questions and the spec, and stay inside its recorded decisions unless the user chooses to change one. This saves the user from explaining what is already written down.

## 2. Interview

- Ask one question at a time. Several at once are hard to answer well.
- Give your recommended answer with each question, so the user can accept in a word.
- When the code can answer a question, read the code instead of asking.
- Walk every branch of the design until each decision is resolved. Resolve the decisions that others depend on first.
- Confirm the testing seam: the highest existing public interface where the change can be checked. Prefer one seam.

Finish with a short list: the resolved decisions and the draft criteria (AC1, AC2, …). Write the spec only after the user confirms the list.

## 3. Write the spec

```markdown
# <title>

**Spec version:** 1

## Problem
One paragraph: what is wrong or missing, and for whom.

## Acceptance criteria
- AC1: When <trigger>, the <system> shall <response>.
- AC2: <checkable statement, for example "`make check` passes">.

## Out of scope
- What we deliberately will not do.

## Testing
Where the change is checked (the seam) and how.

## Decisions
- Each decision from the interview, with its reason and trade-off.

## Changelog
- v1 (<date>): first version.
```

Criteria rules, because build-it, the PR checklist and review-it all cite them:

- 3–7 criteria, each checkable from the diff, a command or a screenshot.
- Behaviour criteria use EARS: `When <trigger>, the <system> shall <response>` (or `While <state>, …`, `If <unwanted condition>, then …`, or `The <system> shall …` for always-true behaviour). Other criteria are plain checkable statements.
- IDs are stable. Never renumber or reuse an ID across versions. A dropped criterion keeps its ID, struck through, with the version that dropped it.
- No user stories. They add length that cannot be checked against a diff.

Leave out **Testing** when the work has nothing to test. Add other sections (constraints, open questions, links) when the interview produced them. If the tracker has a house issue template for specs or features, fit the spec into it and keep the core sections. Apply every `## Rules` item and pass every `## Gates` item.

## 4. Split large work

If the work will not fit one reviewable PR, split it into child specs. Each child ships on its own, is a full spec with its own criteria IDs, links its parent, and has a `Blocked by:` line (`none` when it can start at once). The parent lists its children in dependency order.

Show the breakdown first: per child, its title, what it delivers and what blocks it. Store nothing until the user approves it.

## 5. Store the spec

Where config says. Default:

- The repo has a code host or tracker: create an issue whose body is the spec (`gh issue create`, or the `## Code Host` commands). Report the reference.
- No code host or tracker: write `docs/specs/<slug>.md` and do not commit it; build-it commits it first on its branch.

**Changing an existing spec:** bump `Spec version`, add a changelog line saying what changed and why, and update it in place. Only plan-it edits specs.

## 6. Hand off

Give the user the spec reference (`#N`, URL or path). If `## Then` is set, call that skill with the reference.
