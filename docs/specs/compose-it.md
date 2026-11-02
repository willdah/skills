# compose-it: an SDLC skill suite you configure for your environment

**Spec version:** 1
**Owner:** Will (`willdah`)

## Problem

AI coding agents follow whatever skills happen to be installed, with no shared path from requirement to reviewed PR. Fixed skill chains (one person's favourite grilling, TDD, PR and review skills) do not fit a different team, and enterprise teams have engineering, compliance and review rules that generic skills ignore. Users need a small set of SDLC orchestrators that give sound default guidance and let a solo developer or an enterprise plug in their own skills, rules and gates.

## Acceptance criteria

- [ ] The plugin `compose-it` installs from the public GitHub repo `willdah/compose-it` with `/plugin marketplace add willdah/compose-it`.
- [ ] It ships five skills: `configure-it`, `plan-it`, `build-it`, `stage-it`, `review-it`.
- [ ] Each skill resolves config from `docs/sdlc/<phase>.md` (repo), then `~/.config/sdlc/<phase>.md` (user), then built-in defaults, and states what it loaded and which skills it resolved.
- [ ] Config files use four optional headings: **Use** and **Then** (repo wins over user), **Rules** and **Gates** (both layers apply; user config cannot drop a repo rule or gate). Other text is read as context.
- [ ] A missing default skill gives a built-in fallback and a one-line notice, never a stop. An existing house template (for example `.github/pull_request_template.md`) beats the built-in default.
- [ ] `configure-it` (user-invoked only) reads the repo's existing conventions (forge remote, CI, test and lint commands, PR and issue templates, `CONTRIBUTING.md`, `CODEOWNERS`, `CLAUDE.md`/`AGENTS.md`, installed skills), proposes each phase file in turn, writes only what differs from the defaults, and asks repo or user layer.
- [ ] `plan-it` always runs a grilling session first, then writes a spec with **Problem**, **Acceptance criteria**, **Out of scope**, the decisions made, a `Spec version: N` line and a changelog. The spec goes to the configured tracker as an issue; with no tracker, to `docs/specs/<slug>.md`. Large work splits into a parent spec plus child specs.
- [ ] `build-it` works only against the spec's criteria and out-of-scope list, and records the spec version. New non-trivial logic or a bug fix gets a test that fails without the change; a bug's failing test is committed first. The branch follows Conventional Branch (`<type>/<ticket>-<slug>`) with the same type as the conventional commits. It does not hand off until every gate is green and every criterion is met or marked not met with a reason. It also accepts a review as input and fixes Must fix items and in-scope Should fix items, replying "fixed in `<sha>`" or "skipped: <reason>".
- [ ] `stage-it` runs gates (stops on red), pushes, and opens a draft PR, or updates the existing PR for the branch. The body follows the house template if one exists, otherwise Summary visual / Evidence / Merge Danger, and adds `Closes #N`, "Built against spec vN" and the criteria checklist checked against the diff.
- [ ] `review-it` reports only and never edits code: a PR review with inline comments when a PR exists, a terminal report otherwise. It sorts findings as Must fix / Should fix / Nice to have, and repo rule violations are always Must fix. It flags a stale spec version. It is the only skill that marks a PR ready, only with no open Must fix and every criterion ticked, and config can forbid it.
- [ ] Chaining runs only through **Then** lines. Loop rounds are configurable (default 1) and stop early when there are no Must fix findings or the findings repeat.
- [ ] No skill ever merges a PR, whatever the config says.
- [ ] Forge commands come from config, with GitHub (`gh`) as the default.
- [ ] Every skill folder has a `CREDITS.md`, and the repo root has one: the owner (Will, `willdah`), the repo and the license; each leveraged default skill with its owner and source, marked "optional default, not bundled"; and adapted content with its origin. The same credits appear in each `SKILL.md` frontmatter under `metadata.credits`.
- [ ] `docs/config.md` documents the headings and layering, with a solo example and an enterprise example. The license is MIT.

## Out of scope

- An eval suite or automated tests for the skills.
- An umbrella "run everything" skill; chaining is config.
- Bundling or vendoring other people's skills.
- Merging PRs.
- Migrating the toolkit (closing toolkit PR #9, installing compose-it, moving the Forgejo `gh`→`tea` table and worktree rule into `~/.config/sdlc/`) is follow-up work in the toolkit repo.

## Decisions

- **Config is prose, not YAML.** The reader is a model and enterprise rules are prose. Trade-off: no validation, so each skill states the config and skills it resolved.
- **Four separate skills plus `configure-it`, with no driver skill.** Each skill runs alone (for example reviewing a colleague's PR). Without a spec reference, a skill drafts 3–5 criteria and asks for confirmation.
- **Repo wins on Use/Then; Rules/Gates add up.** A company rule cannot be dropped by personal config; to change it, edit the repo file where others can see it.
- **`plan-it` is prescriptive: grill, then spec.** Both steps can be swapped through **Use**; the order is fixed.
- **Specs version themselves in the issue body.** Trade-off: an edit without a version bump goes unseen; only `plan-it` writes specs.
- **Review reports and build fixes.** Safe default for enterprises where the author's agent must not silently fix and approve its own work.
- **Draft until `review-it` (or a human, per config) says ready.**
- **The "failing test for new logic" rule is fixed; the method (`tdd`) is a default.** Opting out is explicit, through the spec's out-of-scope list.
- **Built for other users from day one:** public GitHub repo with its own marketplace; default skills are soft references with built-in fallbacks.
- **Shared rules repeat briefly in each `SKILL.md`** so every skill works when loaded alone.

### Default Use skills (credited, not bundled)

| Phase | Default | Owner / source |
| --- | --- | --- |
| plan | `grilling`, `to-spec`, `to-tickets` | Matt Pocock, `mattpocock/skills` |
| build | `ponytail` (full) | Dietrich Gebert, `DietrichGebert/ponytail` |
| build | `tdd` | Matt Pocock, `mattpocock/skills` |
| stage | built-in template adapted from `show-me` | Dex Horthy, HumanLayer (`humanlayer/humanlayer`) |
| review | `ponytail-review` | Dietrich Gebert, `DietrichGebert/ponytail` |

## Layout

```text
compose-it/
├── .claude-plugin/{plugin.json,marketplace.json}
├── skills/{configure-it,plan-it,build-it,stage-it,review-it}/{SKILL.md,CREDITS.md}
├── docs/config.md
├── docs/specs/compose-it.md
├── README.md
├── CREDITS.md
└── LICENSE
```

## Changelog

- v1 (2026-11-02): first version, from the grilling session.
