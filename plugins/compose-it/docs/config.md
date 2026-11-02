# Configuring compose-it

Each compose-it skill reads a short markdown file for its phase. Run `/configure-it` to draft them from your repo, or write them by hand.

| Phase | Skill | File |
| --- | --- | --- |
| plan | `plan-it` | `plan.md` |
| build | `build-it` | `build.md` |
| stage | `stage-it` | `stage.md` |
| review | `review-it` | `review.md` |

## Where the files live

1. **Repo:** `docs/sdlc/<phase>.md`, committed, shared by the team.
2. **User:** `~/.config/sdlc/<phase>.md`, your own defaults for every repo.
3. **Built-in:** each skill's defaults, used for anything neither file sets.

Every file is optional. With none, the skills use their defaults and say so once.

## The four headings

All four are optional. Write them as plain markdown lists.

| Heading | Means | Repo and user both set it |
| --- | --- | --- |
| `## Use` | Skills to call for this phase, by job, replacing that job's default | Repo wins |
| `## Rules` | Requirements the work must meet | Both apply |
| `## Gates` | Checks that must pass before the phase is done (commands preferred) | Both apply |
| `## Then` | The skill to chain into when this phase is done | Repo wins |

Write each `## Use` item as `- <job>: <skill>`, for example `- tests: our-tdd`. It replaces the default for that job; jobs you do not name keep their default. The jobs are: plan `grill`, `spec`, `split`; build `approach`, `tests`; review `review` (review also takes extra jobs, for example `- compliance: our-sox-check`). stage-it has no `## Use`: its PR template is built in, and a house template replaces it.

A user file can add rules and gates, but never remove or weaken a repo rule or gate. To change a team rule, edit the repo file where the team can see it.

Text under any other heading is read as context, for example notes on your forge or tracker.

### Forge and tracker

The skills use `gh` (GitHub) by default. For another forge or tracker, describe its commands in prose under a `## Forge` heading in `stage.md`. Every skill that talks to the forge reads it. Where specs go is set in `plan.md` (default: an issue on the forge; with no forge, `docs/specs/<slug>.md`).

### Chaining and loops

`## Then` names the next skill, for example `- stage-it`. In `review.md`, `- build-it, up to 3 rounds` sets the fix loop. The default is 1 round. A loop also stops early when a review finds no Must fix, or finds the same Must fix items as the last round.

## What config cannot change

- No skill merges or approves a PR, and only review-it marks one ready.
- `review-it` never edits code.
- `build-it` gives new non-trivial logic and bug fixes a test that fails without the change, unless the spec's Out of scope says otherwise.
- `build-it` and `stage-it` do not hand off while a gate is red.

## Example: solo developer

`~/.config/sdlc/build.md`

```markdown
## Rules
- Work in a git worktree next to the main clone, not a branch checkout.

## Then
- stage-it
```

`~/.config/sdlc/stage.md`

```markdown
## Then
- review-it
```

`~/.config/sdlc/review.md`

```markdown
## Then
- build-it, up to 2 rounds
```

`~/.config/sdlc/stage.md` can also carry a `## Forge` section, for example a table that maps `gh` commands to `tea` for a Forgejo server.

## Example: enterprise team

`docs/sdlc/plan.md`

```markdown
## Use
- grill: our-discovery-interview
- spec: our-spec-writer

## Rules
- Specs go to Jira project PAY with `jira issue create`; reference them as `PAY-123`.
- Every spec names a data-classification level.
```

`docs/sdlc/build.md`

```markdown
## Rules
- Every DB migration ships with a rollback migration.
- No new dependency without an entry in DEPENDENCIES.md and a licence check.
- Branches are named `<type>/PAY-<n>-<slug>`.

## Gates
- `make check`
- `make sast`
```

`docs/sdlc/review.md`

```markdown
## Use
- review: our-secure-review

## Rules
- Changes under `payments/` need the SOX checklist in the PR body.
- A human marks PRs ready; review-it never does.
```

A developer's personal `~/.config/sdlc/build.md` still applies on top: their rules and gates are added; the team's stay.
