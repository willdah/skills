---
name: review-it
description: "Review a change against its spec and the environment's compliance and review rules, and report findings as Must fix / Should fix / Nice to have, as a PR review or a local report. Never edits code. The only compose-it skill that marks a PR ready. Use when the user wants a change, branch or PR reviewed."
metadata:
  credits:
    uses:
      - skill: ponytail-review
        author: Dietrich Gebert
        source: https://github.com/DietrichGebert/ponytail
        note: optional default, not bundled
---

Review a change and report what must change before it ships. You report; you do not fix.

## 0. Load config

Read these files if they exist: `docs/compose-it/review.md` in the repo, then `~/.config/compose-it/review.md`. Also read `## Forge` from `docs/compose-it/stage.md`, or from `~/.config/compose-it/stage.md` if the repo file has none.

- `## Use`, `## Then`: the repo file wins; else the user file; else the defaults below.
- `## Rules`, `## Gates`: apply the items from both files. A user file cannot remove or weaken a repo rule or gate.
- Any other text is context.

Say in one line what you loaded and which skills you will use, for example `config: repo docs/compose-it/review.md, user none · using: review=ponytail:ponytail-review`. With no files, say once: `no compose-it config; using defaults (run /configure-it to tailor)`.

For each skill you will use that is not installed, say `<skill> not installed; using built-in guidance` and follow the built-in fallback. Never stop because a skill is missing.

To call a skill, use the Skill tool with the name the available-skills list shows for it. Plugin skills carry their plugin's prefix, for example `ponytail:ponytail` or `compose-it:review-it`. A job name such as **tests** is a slot, not a skill: call the skill you resolved for it.

Default, by job: **review** `ponytail:ponytail-review`. Forge `gh`. A `## Use` item replaces the default for the job it names (`- review: our-secure-review`); a team review skill can also be added as a second job (`- compliance: our-sox-check`).

**Config cannot change:** you never edit code, commit or push; you never approve or merge.

## 1. What to review

What the user names: a PR (`#N`, URL), a branch, or files. Nothing named: the current branch against its base, or uncommitted changes if there are any. The input may carry `round k of N` from a chained loop.

Find the spec (PR body, the `Spec:` commit trailer, branch name, or reference). Note its version. If the PR or trailer says it was built against spec vN and the spec is now newer, report that as a **Must fix**: the work may not match the current spec.

No spec: write 3–5 acceptance criteria from the PR description and commits, show them, and continue only when the user confirms. Without confirmed criteria, never mark the PR ready.

## 2. Review

Call the Skill tool with the skill you resolved for **review** in step 0 (default `ponytail:ponytail-review`) on the diff, then the skill for each extra job (for example **compliance**). Fallback for **review**: read the diff and the code it touches (callers, tests); look for, in order, bugs and broken callers, security holes and data loss, what breaks at the repo's expected load, risky logic with no test, real slowness, and code that should not exist. Every finding needs a concrete case that goes wrong.

Then check, yourself:

- Each acceptance criterion against the actual diff.
- Each `## Rules` item. A broken rule is always a **Must fix**.
- Each `## Gates` item: run it. A red gate is a **Must fix**. Do not run a gate that writes files (a formatter without its check flag, `--fix`, `--write`); run its check-only form, or report it as not run.

Sort every finding as **Must fix** (bug, security, data loss, broken rule, red gate, unmet criterion), **Should fix** (risky code without a test, real slowness, duplication, code that should not exist) or **Nice to have**. Number them across groups. Each finding: what it is, the problem, the smallest fix, and what happens if it is skipped.

## 3. Report

- A PR exists: post one review on it. Put each finding as an inline comment on its line where you can, and the full numbered list in the review body (`gh api repos/{owner}/{repo}/pulls/N/reviews`, or the `## Forge` equivalent). Use the comment event unless `## Rules` says to request changes on a Must fix. Never approve.
- No PR: print the report in the terminal.

## 4. Ready or not

Mark the PR ready (`gh pr ready N`, or the `## Forge` equivalent) only when all of these hold:

- No **Must fix** is open.
- There is at least one confirmed acceptance criterion, and every one is met.
- No `## Rules` item says a human marks PRs ready.

Otherwise leave it as a draft and say what blocks it.

## 5. Loop

`## Then` (usually `build-it, up to N rounds`; with no number, N is 1) sets N fix rounds. Input with no round means no fix has run yet: k = 0. Input `round k of N` means fix round k has just run.

If there are open **Must fix** findings, call the `## Then` skill with the PR and `round k+1 of N`. Stop instead, and tell the user why, when any of these holds:

- No **Must fix** is open.
- k = N: every fix round is used.
- k ≥ 1 and the **Must fix** findings are the same as in the previous review-it review on this PR: the fix loop is not making progress.

Give the user the PR link and the verdict.
