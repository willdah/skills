---
name: review-it
description: "Review a change on four axes (spec, the repo's standards, security, compliance) in a fresh context, and report findings as Must fix / Should fix / Nice to have, as a PR review or a local report. Never edits code. The only create-it skill that marks a PR ready. Use when the user wants a change, branch or PR reviewed, a security or compliance check of a diff, or asks whether something is ready to merge."
metadata:
  credits:
    inspired_by:
      - skill: ponytail-review
        author: Dietrich Gebert
        source: https://github.com/DietrichGebert/ponytail
        took: read the connected code; the bug, risk, scale, test, speed, lean order; no finding without a concrete case; the four-part finding; verdict and not-checked lines
      - skill: code-review
        author: Matt Pocock
        source: https://github.com/mattpocock/skills
        took: separate spec and standards axes; review in a separate context
      - skill: caveman
        author: Julius Brussee
        source: https://github.com/JuliusBrussee/caveman
        took: terse tone for agent chat
---

Review a change and report what must change before it ships. You report; you do not fix.

**Called by create-it** (arguments start `called by create-it:`): do steps 1–4, skip step 5 (create-it runs the loop), and return the verdict: open Must fix findings, whether the spec is stale, and whether you marked the PR ready.

## Tone

Agent chat (status, verdict): terse, fragments OK. Findings: very simple English, short full sentences, explain a technical term the first time; the reader may never have seen this code. Security warnings: normal prose.

## 0. Load config

Read if present: `docs/create-it/review.md` (repo), then `~/.config/create-it/review.md` (user). Also read `## Code Host` from `docs/create-it/stage.md`, else from `~/.config/create-it/stage.md`.

- `## Then`, `## Code Host`: the repo value wins, else the user value, else the defaults (code host `gh`).
- `## Rules`, `## Gates`: apply the items from both files. A user file cannot drop or weaken a repo item. A Rules item may name another skill to run as an extra check (for example `Also run the our-sox-check skill on changes under payments/`): run it and fold its findings into the Compliance axis.
- Any other text is context.

Say in one line what you loaded, for example `config: repo docs/create-it/review.md, user none`. With no files: `no create-it config; using defaults (run /explore-it to tailor)`.

**Config cannot change:** you never edit code, commit or push; you never approve or merge.

## 1. What to review

What the user names: a PR (`#N`, URL), a branch, or files. Nothing named: the current branch against its base, or uncommitted changes if there are any. The input may carry `round k of N` from a chained loop.

Find the spec (PR body, the `Spec:` commit trailer, branch name, or reference). Note its version. If the PR or trailer says it was built against spec vN and the spec is now newer, put **Stale spec: built against vN, spec is now vM** at the top of the report. It blocks ready (step 4) but is not a **Must fix**, because a fix round cannot change the spec: the user decides whether to rebuild.

No spec: write 3–5 criteria (IDs `AC1`…) from the PR description and commits, show them, and continue only when the user confirms. Without confirmed criteria, never mark the PR ready.

Collect the sources the axes need: the diff, the spec, the repo's standards docs (`CONTRIBUTING.md`, `CODING_STANDARDS.md`, `AGENTS.md`, `CLAUDE.md`, style guides), its compliance sources (`## Rules`, `SECURITY.md`, compliance or policy docs, checklists in the PR template, `CODEOWNERS` entries on sensitive paths), and `ENV_MAP.md` if present.

## 2. Review in a fresh context

Where your agent can run a subagent, hand steps 2a–2c to one, with the diff command, the spec, the source list above and this section. The reviewer then sees the change the way a colleague would, without the author's reasoning, which matters most when create-it chained build and review in one session. Without subagents, review here.

### 2a. Understand first

Read the diff, then the code it touches: callers of every changed function, the functions it calls, the tests. When a signature, return value or behaviour changes, find every caller. Find the expected load in the repo (README, deploy config) and say which load you assumed.

### 2b. Four axes

**Spec.** Each criterion by ID against the actual diff: missing, partial (`AC3 partial`), or done wrong. Behaviour nobody asked for, including anything Out of scope excludes.

**Standards.** In this order: bugs (wrong result, crash, missed edge case, a broken caller, a fix in one caller while the shared code stays broken); scale (races, work repeated per process, unbounded growth, a query per item, per-process state that must be shared); risky logic with no test that fails when it breaks; real slowness; code that should not exist (dead code, unused options, a helper the repo already has, an abstraction with one use, near-copies that must change together). Also breaks of the repo's documented standards, each citing file and rule. A `shortcut:` comment that names its limit is a decision, not a finding, unless the expected load already crosses it.

**Security.** Check the diff against the OWASP Top 10 categories. Cite a CWE ID on each finding (for example `CWE-89`):

- Injection and unvalidated input at trust boundaries: SQL, shell, path traversal, SSRF, unsafe deserialization.
- Broken authentication or authorization: missing checks, one user's or tenant's data reaching another.
- Secrets in code, and secrets or personal data in logs, errors or URLs.
- Weak cryptography: weak randomness, home-made crypto, disabled TLS verification.
- New dependencies: known vulnerabilities, licence, whether the package is maintained.

**Compliance.** Each documented rule from the compliance sources against the diff, citing the rule. With none found, report `Compliance: no documented rules found`. Do not invent a framework (SOX, HIPAA, PCI) the repo does not name.

### 2c. Check before you report

- Every finding needs a concrete case: "this input or situation leads to this wrong result". For security, a concrete way to exploit it. No case, no finding.
- Re-read the lines and confirm: the caller exists, the value can really be empty, the code really is unused.
- Propose the smallest fix that works. Prefer fixes that delete code.
- No style taste, no "consider", no vague worries.

### 2d. Gates

Run each `## Gates` item yourself. Do not run a gate that writes files (a formatter without its check flag, `--fix`, `--write`); run its check-only form, or report it as not run.

## 3. Sort and report

Sort every finding into one list:

- **Must fix:** bug, data loss, breaks at the expected load, a broken `## Rules` item, a red gate, an unmet criterion, an exploitable security finding, any compliance finding.
- **Should fix:** risky code without a test, real slowness, duplication, code that should not exist, security hardening with no concrete exploit.
- **Nice to have:** small speed-ups, shorter forms.

Number findings across groups so the user can say "fix 2 and 5". Tag each with its axis. Every finding has four parts, each one or two short sentences:

```markdown
2. **[Security] User id from the URL reaches SQL unescaped** (`api/orders.py:L40-44`, CWE-89)
   - **What this is:** The orders endpoint looks up orders for the id in the URL.
   - **Problem:** The id is pasted into the SQL string. `?id=1 OR 1=1` returns every customer's orders.
   - **Fix:** Pass the id as a query parameter: `cur.execute("… WHERE user_id = %s", (uid,))`.
   - **If we skip it:** Anyone can read every order in the database.
```

The report opens with `What this change does:` in two or three sentences, and ends with:

- `Verdict: Ship.` or `Verdict: fix 1 and 3 first.`
- `Not checked:` one line, if something mattered and you could not check it.

Nothing found: `What this change does:`, then `Looks good. Ship.` and one line on what you checked.

Where it goes:

- A PR exists: post one review. Put each finding as an inline comment on its line where you can, and the full numbered list in the review body (`gh api repos/{owner}/{repo}/pulls/N/reviews`, or the `## Code Host` equivalent). Use the comment event unless `## Rules` says to request changes on a Must fix. Never approve.
- No PR: print the report in the terminal.

## 4. Ready or not

Mark the PR ready (`gh pr ready N`, or the `## Code Host` equivalent) only when all of these hold:

- No **Must fix** is open.
- The spec is not stale.
- There is at least one confirmed criterion, and every one is met.
- No `## Rules` item says a human marks PRs ready.

Otherwise leave it as a draft and say what blocks it.

## 5. Loop

`## Then` (usually `build-it, up to N rounds`; with no number, N is 1) sets N fix rounds. Input with no round means no fix has run yet: k = 0. Input `round k of N` means fix round k has just run.

If there are open **Must fix** findings, call the `## Then` skill with the PR and `round k+1 of N`. Stop instead, and tell the user why, when any of these holds:

- No **Must fix** is open.
- k = N: every fix round is used.
- k ≥ 1 and the **Must fix** findings are the same as in the previous review-it review on this PR: the loop is not making progress.

Give the user the PR link and the verdict.
