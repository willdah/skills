---
name: plan-it
description: "Plan a piece of work: grill the user until requirements are resolved, then write a versioned spec (problem, acceptance criteria, out of scope, decisions) that build-it, stage-it and review-it check against. Use when the user wants to plan, scope, spec or nail down requirements for a feature, fix or change."
metadata:
  credits:
    owner: "Will (willdah), https://github.com/willdah/skills, MIT"
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

Read these files if they exist: `docs/sdlc/plan.md` in the repo, then `~/.config/sdlc/plan.md`. Also read `## Forge` in `docs/sdlc/stage.md` or `~/.config/sdlc/stage.md` (repo first).

- `## Use`, `## Then`: the repo file wins; else the user file; else the defaults below.
- `## Rules`: apply the items from both files. A user file cannot remove or weaken a repo rule.
- Any other text is context.

Say in one line what you loaded and which skills you will use, for example `config: repo docs/sdlc/plan.md, user none · using: grilling, to-spec`. With no files, say once: `no compose-it config; using defaults (run /configure-it to tailor)`.

For each skill you will use that is not installed, say `<skill> not installed; using built-in guidance` and follow the built-in fallback. Never stop because a skill is missing.

Defaults: grill with `grilling`; write the spec with `to-spec`; split large work with `to-tickets`.

## 1. Grill

Call the grill skill on the user's request. Fallback: interview the user one question at a time; give your recommended answer with each question; explore the codebase instead of asking when the code can answer; walk every branch of the design until each decision is resolved.

Do not write the spec until the user agrees the open questions are resolved.

## 2. Write the spec

Call the spec skill on the resolved session, but make sure the result has this core. Fallback: write it yourself.

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

Add other sections (constraints, open questions, links) when the grilling produced them. Apply every `## Rules` item, for example a required field.

**Large work:** if the work will not fit one reviewable PR, split it with the split skill (fallback: one child spec per slice that can ship on its own). Each child has the full core and links its parent; the parent lists its children.

## 3. Store the spec

Where config says. Default:

- The repo has a forge or tracker: create an issue whose body is the spec (`gh issue create`, or the `## Forge` commands). Report the issue reference.
- No forge or tracker: write `docs/specs/<slug>.md` and do not commit it; `build-it` commits it first on its branch.

**Changing an existing spec:** bump `Spec version`, add a changelog line saying what changed and why, and update it in place. Only plan-it edits specs.

## 4. Hand off

Give the user the spec reference (`#N`, URL or path). If `## Then` is set, call that skill with the reference.
