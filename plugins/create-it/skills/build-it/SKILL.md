---
name: build-it
description: "Build one spec, ticket or described task on a branch until its acceptance criteria are met and its gates are green, with the smallest complete change and test-first logic, following the repo's and user's create-it config. Also fixes the findings of a review. Use when the user wants to implement, build or fix something, or to address review findings on a PR."
metadata:
  credits:
    inspired_by:
      - skill: ponytail
        author: Dietrich Gebert
        source: https://github.com/DietrichGebert/ponytail
        took: list every place the change reaches; the first-option-that-works ladder; never-cut list; shortcut comments; report what was skipped
      - skill: tdd
        author: Matt Pocock
        source: https://github.com/mattpocock/skills
        took: test at the agreed seam through public interfaces; red before green; implementation-coupled and tautological anti-patterns
      - skill: caveman
        author: Julius Brussee
        source: https://github.com/JuliusBrussee/caveman
        took: terse tone for agent chat
---

Build the work the user names against its spec, then hand off. Invoking build-it is approval to create a branch and commit.

**Called by create-it** (arguments start `called by create-it:`): do this phase, chain to nothing (skip `## Then`, and skip step 7 of Fixing a review), and return the branch, each criterion as met or not met, and any assumptions. Do not ask the user: where the spec is silent, choose, and report the choice as an assumption.

## Tone

Agent chat (status, progress, hand-off): terse, fragments OK. Commit messages and replies on review findings: short full sentences. Warnings about destructive or irreversible steps: normal prose.

## 0. Load config

Read if present: `docs/create-it/build.md` (repo), then `~/.config/create-it/build.md` (user). Also read `## Code Host` from `docs/create-it/stage.md`, else from `~/.config/create-it/stage.md`.

- `## Then`, `## Code Host`: the repo value wins, else the user value, else the defaults (code host `gh`).
- `## Rules`, `## Gates`: apply the items from both files. A user file cannot drop or weaken a repo item.
- Any other text is context.

Say in one line what you loaded, for example `config: repo none, user ~/.config/create-it/build.md`. With no files: `no create-it config; using defaults (run /explore-it to tailor)`.

**Config cannot change:** you never merge a PR; new non-trivial logic and bug fixes get a test that fails without the change (unless the spec's Out of scope excludes tests); you do not hand off while a gate is red.

## 1. Get the spec

- `#N`, a URL or a tracker key: fetch it and state its title and `Spec version`.
- A file path: read it. If it is not committed, commit it first on the branch (step 2).
- A review on a PR (for example `address review on #9`): go to **Fixing a review**.
- Plain text, no spec: write 3–5 criteria (IDs `AC1`…, EARS for behaviour) and an out-of-scope list, show them, and start only when the user confirms.
- An ambiguous reference: ask.

Work only to the acceptance criteria. Do not build what Out of scope excludes.

Record the spec in the first commit on the branch as a trailer: `Spec: <reference> v<N>` (for example `Spec: #42 v3`). stage-it and review-it read it to detect a spec that changed after the build.

## 2. Branch

Take the change type from the spec once: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `ci`, `perf`. The branch name and the main commit type both use it.

- Default branch name: `<type>/<ticket>-<slug>` (Conventional Branch), or `<type>/<slug>` with no ticket. A `## Rules` naming scheme replaces it.
- On the default branch: create the branch from the up-to-date default branch. Use a worktree instead if config says so.
- Already on a feature branch for this work: stay there.
- A new worktree or clone has only tracked files: install dependencies and copy the local config it needs.

## 3. Build

**Scope first.** Read the code the change touches, then list every place it must reach: callers, tests, fixtures, config, exports. That list is the scope; extra features are not. A change that misses one caller is unfinished.

**Take the first option that fully works**, in this order:

1. Not needed: skip options, features and flexibility nobody asked for.
2. Already in the codebase (a helper, component, pattern): use it the way the surrounding code does.
3. The standard library or platform does it.
4. An installed dependency does it. Never add a dependency for a few lines.
5. Otherwise the least new code that works.

Add no abstraction, wrapper, option or "for later" code nobody asked for; each one is code someone must read, test and fix later. Keep the structure and conventions the codebase already has. For a bug fix, find every caller of the function you touch and fix the root cause once, in the shared code. Code you move keeps its error handling and validation.

**Never cut:** validation at trust boundaries, error handling that prevents data loss, security, accessibility, anything the spec asks for. A shortcut with a known limit gets a comment: `shortcut: <the limit>, <when to upgrade>`.

**Tests.** New non-trivial logic (a branch, loop, parser, money, security, a new script) and bug fixes get a test that fails without the change.

- Test at the seam in the spec's **Testing** section, through public interfaces. A test that reaches inside breaks on every refactor without catching real bugs.
- Red before green: write one failing test, write only enough code to pass it, then the next test. Writing all tests first locks in imagined behaviour.
- For a bug fix, commit the failing test that reproduces the bug before the fix.
- Avoid two anti-patterns: *implementation-coupled* tests (mocking internal collaborators, testing private methods, checking through a side channel) and *tautological* tests (computing the expected value the way the code does). Expected values come from the spec, a worked example or a known-good literal.
- Config, wiring, docs and renames need no test.

Apply every `## Rules` item. Run typechecking and single test files as you go.

Commits: conventional commits, one per logical step, unless `## Rules` sets another format.

## 4. Check before hand-off

- Run every `## Gates` item. With no gates, find the obvious checks (CI config, `make check`, `npm test`, `pytest`, `cargo test`), run them, and say which you chose.
- Run the full test suite once.
- Check each criterion by ID against the actual diff: met, or not met with a reason.

A red gate: fix it. If you cannot, stop and tell the user; do not hand off.

## 5. Hand off

Tell the user: the branch, the spec version, each criterion by ID as met or not met, and one or two lines on what you skipped or did not check and any risk. stage-it puts those lines in the PR's Follow-ups. If `## Then` is set, call that skill with the spec reference and the branch.

## Fixing a review

Input is a review on a PR, optionally with `round k of N` from a chained loop.

1. Fetch the review's findings (Must fix / Should fix / Nice to have) and the spec.
2. Check out the PR's branch.
3. Fix every **Must fix**. Fix a **Should fix** when the fix stays inside code this PR changes. Skip **Nice to have**. The method in step 3 applies to every fix.
4. Commit the fixes (conventional commits), run the gates (step 4), push.
5. Reply on each finding: `fixed in <sha>` or `skipped: <reason>`.
6. Update the PR body's criteria checklist if the fixes changed it.
7. Invoke `review-it` directly with the PR and the same `round k of N`. Do not follow `## Then` here: the loop belongs to review.md, and a skill in between would lose the round.
