---
name: plan-it
description: "Plan a piece of work: grill the user until requirements are resolved, then write a versioned spec (problem, acceptance criteria, out of scope, decisions) that build-it, stage-it and review-it check against. Use when the user wants to plan, scope, spec or nail down requirements for a feature, fix or change."
metadata:
  credits:
    uses:
      - skill: grilling
        author: Matt Pocock
        source: https://github.com/mattpocock/skills
        note: optional default, not bundled
      - skill: to-spec
        author: Matt Pocock
        source: https://github.com/mattpocock/skills
        note: optional default, not bundled
      - skill: to-tickets
        author: Matt Pocock
        source: https://github.com/mattpocock/skills
        note: optional default, not bundled
---

Turn what the user wants into a spec the other compose-it skills can check work against. Always in this order: grill, then write the spec.

## 0. Load config

Read these files if they exist: `docs/compose-it/plan.md` in the repo, then `~/.config/compose-it/plan.md`. Also read `## Forge` from `docs/compose-it/stage.md`, or from `~/.config/compose-it/stage.md` if the repo file has none.

- `## Use`, `## Then`: the repo file wins; else the user file; else the defaults below.
- `## Rules`, `## Gates`: apply the items from both files. A user file cannot remove or weaken a repo rule or gate. Gates here are checks the spec must pass before hand-off.
- Any other text is context.

Say in one line what you loaded and which skills you will use, for example `config: repo docs/compose-it/plan.md, user none · using: grill=grilling, spec=to-spec, split=to-tickets`. With no files, say once: `no compose-it config; using defaults (run /configure-it to tailor)`.

For each skill you will use that is not installed, say `<skill> not installed; using built-in guidance` and follow the built-in fallback. Never stop because a skill is missing.

To call a skill, invoke it the way your agent loads skills (in Claude Code, the Skill tool), by the name your agent lists it under. Defaults below use bare names; some agents add a plugin prefix (Claude Code lists `ponytail` as `ponytail`). A job name such as **tests** is a slot, not a skill: call the skill you resolved for it.

Defaults, by job: **grill** `grilling`; **spec** `to-spec`; **split** `to-tickets`. A `## Use` item replaces the default for the job it names (`- spec: our-spec-writer`); jobs it does not name keep their default.

**Config cannot change:** you never merge a PR; you always grill before writing the spec.

## 1. Grill

Invoke the skill you resolved for **grill** in step 0 (default `grilling`) on the user's request. Fallback: interview the user one question at a time; give your recommended answer with each question; explore the codebase instead of asking when the code can answer; walk every branch of the design until each decision is resolved.

Do not write the spec until the user agrees the open questions are resolved.

## 2. Write the spec

Invoke the skill you resolved for **spec** in step 0 (default `to-spec`) on the resolved session, but make sure the result has this core. Fallback: write it yourself.

```markdown
# <title>

**Spec version:** 1

## Problem
One paragraph: what is wrong or missing, and for whom.

## Acceptance criteria
- [ ] 3–7 statements, each one checkable from the diff, a command or a screenshot.

## Out of scope
- What we deliberately will not do.

## Decisions
- Each decision from the grilling, with its reason and trade-off.

## Changelog
- v1 (<date>): first version.
```

Add other sections (constraints, open questions, links) when the grilling produced them. If the tracker has a house issue template for specs or features, fit the spec into it and keep the core sections. Apply every `## Rules` item, for example a required field, and pass every `## Gates` item.

**Large work:** if the work will not fit one reviewable PR, split it: invoke the skill you resolved for **split** in step 0 (default `to-tickets`) (fallback: one child spec per slice that can ship on its own). Each child has the full core and links its parent; the parent lists its children.

## 3. Store the spec

Where config says. Default:

- The repo has a forge or tracker: create an issue whose body is the spec (`gh issue create`, or the `## Forge` commands). Report the issue reference.
- No forge or tracker: write `docs/specs/<slug>.md` and do not commit it; `build-it` commits it first on its branch.

**Changing an existing spec:** bump `Spec version`, add a changelog line saying what changed and why, and update it in place. Only plan-it edits specs.

## 4. Hand off

Give the user the spec reference (`#N`, URL or path). If `## Then` is set, call that skill with the reference.
