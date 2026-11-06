# Configuring create-it

Each create-it phase skill reads a short markdown file for its phase. Run `/explore-it` to map your environment and draft them from what it finds, or write them by hand.

| Phase | Skill | File |
| --- | --- | --- |
| plan | `plan-it` | `plan.md` |
| build | `build-it` | `build.md` |
| stage | `stage-it` | `stage.md` |
| review | `review-it` | `review.md` |

## Where the files live

1. **Repo:** `docs/create-it/<phase>.md`, committed, shared by the team.
2. **User:** `~/.config/create-it/<phase>.md`, your own defaults for every repo.
3. **Built-in:** each skill's defaults, used for anything neither file sets.

Every file is optional. With none, the skills use their defaults and say so once.

## The headings

All are optional. Write them as plain markdown lists.

| Heading | Means | Repo and user both set it |
| --- | --- | --- |
| `## Rules` | Requirements the work must meet | Both apply |
| `## Gates` | Checks that must pass before the phase is done (commands preferred) | Both apply |
| `## Then` | The skill to chain into when this phase is done | Repo wins |
| `## Code Host` (in `stage.md` only) | Commands for your code host | Repo wins |

The methods (how to interview, build, write a PR, review) are built into the skills and cannot be swapped. To add a check of your own, write it as a Rules item, for example `- Also run the our-sox-check skill on changes under payments/`. review-it runs a skill named this way and reports its findings under Compliance.

A user file can add rules and gates, but never remove or weaken a repo rule or gate. To change a team rule, edit the repo file where the team can see it.

Text under any other heading is read as context, for example notes on your tracker.

### Code host and tracker

The **code host** is the platform that hosts your repo, pull requests and issues: GitHub, GitLab, Forgejo, Bitbucket. The skills use `gh` (GitHub) by default. For another code host, describe its commands in prose under `## Code Host` in `stage.md`: how to open a draft PR, mark it ready, post a review, and create an issue. Every skill that talks to the code host reads it, from the repo `stage.md` if it has one, else from the user `stage.md`. Where specs go is set in `plan.md` (default: an issue on the code host; with none, `docs/specs/<slug>.md`).

### Chaining and loops

`## Then` names the next skill, for example `- stage-it`. In `review.md`, `- build-it, up to 3 rounds` sets the fix loop. The default is 1 round. A loop also stops early when a review finds no Must fix, or finds the same Must fix items as the last round.

`/create-it` runs plan → build → stage → review in a fixed order and ignores every `## Then` line, except that it takes the round count N from `review.md`.

## What config cannot change

- No skill merges or approves a PR, and only review-it marks one ready.
- `review-it` never edits code.
- `build-it` gives new non-trivial logic and bug fixes a test that fails without the change, unless the spec's Out of scope says otherwise.
- `build-it` and `stage-it` do not hand off while a gate is red.

## Example: solo developer

`~/.config/create-it/build.md`

```markdown
## Rules
- Work in a git worktree next to the main clone, not a branch checkout.

## Then
- stage-it
```

`~/.config/create-it/stage.md`

```markdown
## Code Host
Forgejo at git.example.com. Use `tea` instead of `gh`:
- open a draft PR: `tea pulls create --head <branch> --base main -t "WIP: <title>" --description-file <file>`
- mark ready: edit the title to drop `WIP: `
- post a review: `tea api -X POST 'repos/{owner}/{repo}/pulls/N/reviews' -d @review.json`
- create an issue: `tea issues create -t <title> -d <body>`

## Then
- review-it
```

`~/.config/create-it/review.md`

```markdown
## Then
- build-it, up to 2 rounds
```

## Example: enterprise team

`docs/create-it/plan.md`

```markdown
## Rules
- Specs go to Jira project PAY with `jira issue create`; reference them as `PAY-123`.
- Every spec names a data-classification level.
```

`docs/create-it/build.md`

```markdown
## Rules
- Every DB migration ships with a rollback migration.
- No new dependency without an entry in DEPENDENCIES.md and a license check.
- Branches are named `<type>/PAY-<n>-<slug>`.

## Gates
- `make check`
- `make sast`
- `gitleaks detect --no-banner`
```

`docs/create-it/review.md`

```markdown
## Rules
- Also run the our-sox-check skill on changes under `payments/`.
- Changes under `payments/` need the SOX checklist in the PR body.
- A human marks PRs ready; review-it never does.
```

A developer's personal `~/.config/create-it/build.md` still applies on top: their rules and gates are added; the team's stay.

## Upgrading from 0.2

`## Use` and `## Forge` are no longer read. Delete `## Use` lines, or rewrite a team skill you want run as a Rules item. Rename `## Forge` to `## Code Host`. `configure-it` is replaced by `explore-it`.
