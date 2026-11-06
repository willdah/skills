---
name: stage-it
description: "Stage finished work for review: run the gates, push the branch, and open or update a draft PR whose body points the reviewer at what needs judgment, shows the change visually, gives evidence, verify steps and merge danger, and checks the spec's acceptance criteria by ID against the diff. Use when the user wants to open, create or update a PR or merge request."
metadata:
  credits:
    inspired_by:
      - skill: show-me
        author: Dex Horthy
        organization: HumanLayer
        source: https://github.com/humanlayer/skills
        took: the menu of text visuals, the smallest view that makes the point, diff shape matched to the topic, each visual next to the text it supports
      - skill: pr
        author: Matt Pocock
        source: https://github.com/mattpocock/skills
        took: Evidence and Merge danger sections
      - skill: caveman
        author: Julius Brussee
        source: https://github.com/JuliusBrussee/caveman
        took: terse tone for agent chat
---

Put a branch up for review as a draft PR that a human, or an agent, can review quickly. Invoking stage-it is approval to push and open or update the PR.

**Called by create-it** (arguments start `called by create-it:`): do this phase, chain to nothing (skip `## Then`), add the assumptions it passes to Review focus, and return the PR link.

## Tone

Agent chat (status, hand-off): terse, fragments OK. The PR body: short full sentences, no filler or hedging.

## 0. Load config

Read if present: `docs/create-it/stage.md` (repo), then `~/.config/create-it/stage.md` (user). Also read the `## Gates` of `build.md` at both levels.

- `## Then`, `## Code Host`: the repo value wins, else the user value, else the defaults.
- `## Rules`, `## Gates`: apply the items from both files. A user file cannot drop or weaken a repo item.
- Any other text is context.

Say in one line what you loaded, for example `config: repo docs/create-it/stage.md, user none · template: .github/pull_request_template.md · code host: gh`. With no files: `no create-it config; using defaults (run /explore-it to tailor)`.

Defaults: code host `gh`; open as draft; the built-in template below.

**Config cannot change:** you never merge; you never mark a PR ready; you do not open or update a PR while a gate is red.

## 1. Gates

Run every gate from `stage.md` and `build.md`. Any red: stop, report it, open nothing.

## 2. Push

Push the branch and set its upstream.

## 3. Find the spec

From the `Spec:` commit trailer build-it wrote, the branch name, or the user's reference (`#N`, URL, path). Read its acceptance criteria.

The version for "Built against spec v<N>" comes from the `Spec:` trailer, not from the spec's current version: if the spec changed after the build, review-it must be able to see that. No trailer: use the current version and say so.

No spec: use the criteria build-it reported. None reported: write 3–5 criteria (IDs `AC1`…) from the commits, show them, and continue only when the user confirms.

## 4. Write the body

**House template first.** If the repo has a PR template (`.github/pull_request_template.md`, a file in `.github/PULL_REQUEST_TEMPLATE/`, `pull_request_template.md` under `.gitea/`, `.forgejo/` or `docs/`, or one named in config), fill it in. Otherwise use the built-in template. In both cases add the **Spec** block.

Built-in template:

````markdown
## Review focus

1. [`path/file.ext:L10-24`](link): <why this needs human judgment>

## Summary

<the smallest text visual that shows the change, with one line of context>

## Evidence

- **Before:** <failing test, output or screenshot>
  **After:** <passing test, output or screenshot>

## How to verify

1. <command to run or file to inspect>
2. <what to look for>

## Merge danger

**Door:** <one-way or two-way>: <one line on how hard it is to undo>

**Blast radius:** <one word>: <who or what could break>

## Follow-ups

- <known gap, deliberately left out work, or something not checked>, or "None".
````

**Review focus** comes first because a reviewer's attention is the scarce resource. Name 1–3 places that need human judgment, each with a file link and one line on why: a one-way door, security, an assumption the spec does not cover, a criterion not met, a decision the code makes that a reader could miss. Nothing needs judgment: write "None".

**Summary visual.** Pick the smallest view that makes the change clear:

- A file layout change: a short file tree, as a `diff` when most of the tree exists already.
- A runtime flow change: a call tree, as a `diff` against the old one.
- A UI change: a component tree with the state and module boundaries that matter.
- New logic or an algorithm: pseudocode, as a `diff` when it changes existing logic.
- Several parts talking to each other: a Mermaid sequence or flow diagram.
- Mostly new code the reviewer needs to see whole: the key block itself.

Mark what changed so the eye goes there: `+`/`-` diff markers in trees and pseudocode, `classDef changed` on new or changed Mermaid nodes, a `# ← why` note on the one or two lines that carry a decision. Unchanged context stays plain. Show only the calls, files, props and states that answer "what changed and why". Keep visuals in text, not images, so an agent reviewer reads what the human sees. Put each visual next to the line it supports; one is usually enough.

**Evidence:** screenshots for visual changes, test or command output otherwise. Output beats claims.

**Follow-ups:** include build-it's "skipped or not checked" lines and any risk it reported.

Spec block, always appended:

```markdown
## Spec

Closes #N   <!-- omit with no tracker issue; use the tracker's own syntax when not GitHub -->
Built against spec v<N>

- [x] AC1: <criterion>
- [ ] AC3: <criterion> — not met: <why>
```

Tick each criterion against the actual diff, not against intent. Apply every `## Rules` item, for example a required checklist.

## 5. Open or update

- An open PR exists for this branch: update its body (and title if the scope changed). Do not open a second one.
- Otherwise: open a **draft** PR against the default branch (`gh pr create --draft`, or the `## Code Host` equivalent). If the code host has no drafts, use its work-in-progress marker (for example a `WIP:` title prefix).

Do not mark the PR ready yourself; review-it or a human does that.

## 6. Hand off

Give the user the PR link. If `## Then` is set, call that skill with the PR.
