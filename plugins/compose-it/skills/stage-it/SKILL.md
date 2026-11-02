---
name: stage-it
description: "Stage finished work for review: run the gates, push the branch, and open or update a draft PR whose body shows the change visually, gives evidence and merge danger, and checks the spec's acceptance criteria against the diff. Use when the user wants to open, create or update a PR or merge request."
metadata:
  credits:
    owner: "Will (willdah), https://github.com/willdah/skills, MIT"
    adapted:
      - from: show-me
        author: Dex Horthy
        organisation: HumanLayer
        source: https://github.com/humanlayer/humanlayer
        note: the Summary section's choice of visuals is adapted, not copied
---

Put a branch up for review as a draft PR that a human can review quickly. Invoking stage-it is approval to push and open or update the PR.

## 0. Load config

Read these files if they exist: `docs/sdlc/stage.md` in the repo, then `~/.config/sdlc/stage.md`. Also read the `## Gates` of `build.md` at both levels.

- `## Use`, `## Then`, `## Forge`: the repo file wins; else the user file; else the defaults below.
- `## Rules`, `## Gates`: apply the items from both files. A user file cannot remove or weaken a repo rule or gate.
- Any other text is context.

Say in one line what you loaded, for example `config: repo docs/sdlc/stage.md, user none · template: .github/pull_request_template.md · forge: gh`. With no files, say once: `no compose-it config; using defaults (run /configure-it to tailor)`.

For each skill in `## Use` that is not installed, say `<skill> not installed; using built-in guidance` and continue. Never stop because a skill is missing.

Defaults: forge `gh`; open as draft; the built-in template below.

**Config cannot change:** you never merge; you do not open or update a PR while a gate is red.

## 1. Gates

Run every gate from `stage.md` and `build.md`. Any red: stop, report it, open nothing.

## 2. Push

Push the branch and set its upstream.

## 3. Find the spec

From the branch name, commits or the user's reference (`#N`, URL, path). Read its `Spec version` and acceptance criteria. No spec: use the criteria build-it reported, or ask.

## 4. Write the body

**House template first.** If the repo has a PR template (`.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE/`, `.gitea/`, `.forgejo/`, `docs/`, or one named in config), fill it in. Otherwise use the built-in template. In both cases add the **Spec** block.

Built-in template:

````markdown
## Summary

<the smallest visual that shows the change, see below>

## Evidence

- **Before:** <failing test, output or screenshot>
  **After:** <passing test, output or screenshot>

## Merge danger

**Door:** <one-way or two-way>: <one line on how hard it is to undo>

**Blast radius:** <one word>: <who or what could break>
````

Choose the Summary visual by what the change is:

- A file layout change: a short file tree, as a `diff` when most of the tree exists already.
- A runtime flow change: a call tree, as a `diff` against the old one.
- A UI change: a component tree with the state and module boundaries that matter.
- New logic or an algorithm: pseudocode.
- Several parts talking to each other: a Mermaid sequence or flow diagram.
- Mostly new code that the reviewer needs to see whole: the key block itself.

Show only the calls, files, props and states that answer "what changed and why". One visual is usually enough. Keep prose short. Screenshots beat output for visual changes; output beats claims.

Spec block, always appended:

```markdown
## Spec

Closes #N   <!-- omit with no tracker issue; use the tracker's own syntax when not GitHub -->
Built against spec v<N>

- [x] <criterion met>
- [ ] <criterion not met>: <why>
```

Tick each criterion against the actual diff, not against intent. Apply every `## Rules` item, for example a required checklist.

## 5. Open or update

- An open PR exists for this branch: update its body (and title if the scope changed). Do not open a second one.
- Otherwise: open a **draft** PR against the default branch (`gh pr create --draft`, or the `## Forge` equivalent). A `## Rules` item can say to open it ready instead.

Do not mark the PR ready yourself; review-it or a human does that.

## 6. Hand off

Give the user the PR link. If `## Then` is set, call that skill with the PR.
