# create-it: an SDLC skill suite with proven methods built in

**Spec version:** 8
**Owner:** Will (`willdah`)

## Problem

AI coding agents follow whatever skills happen to be installed, with no shared path from requirement to reviewed PR. Version 7 tried to fix this with pluggable skill slots (`## Use`) and soft calls to other people's skills, with built-in fallbacks. In practice that flexibility gives uneven results that depend on what is installed, and it hides the methods that work behind indirection. Users need one set of SDLC skills with the proven methods built in, a spec format that agents and human reviewers can both check work against, and an environment scan that tailors the rules to their repo, team and tooling.

## Acceptance criteria

- AC1: The plugin `create-it` lives in the public GitHub marketplace repo `willdah/skills` under `plugins/create-it/`, at version `0.3.0`, and installs with `/plugin marketplace add willdah/skills` then `/plugin install create-it@willdah-skills`. The marketplace has no standalone `explorer` entry.
- AC2: The plugin ships six skills: `create-it` (the driver), `explore-it`, `plan-it`, `build-it`, `stage-it`, `review-it`. `configure-it` and the top-level `skills/explorer/` are removed.
- AC3: No skill calls another author's skill, and no skill reads a `## Use` heading. Config files take `## Rules`, `## Gates`, `## Then`, plus `## Code Host` in `stage.md`. Each skill resolves config from `docs/create-it/<phase>.md` (repo), then `~/.config/create-it/<phase>.md` (user), then built-in defaults, and states in one line what it loaded. Rules and Gates add up across layers; a user file cannot drop or weaken a repo item; Then and Code Host take the repo value first.
- AC4: When `plan-it` runs, it first reads the repo's domain docs (for example a glossary or `CONTEXT.md`, ADRs, architecture docs, `AGENTS.md`/`CLAUDE.md`, `CONTRIBUTING.md`, the README, `ENV_MAP.md`), then interviews the user one question at a time with a recommended answer each, explores the code instead of asking where the code can answer, confirms the testing seam, and ends with a list of resolved decisions and draft criteria that the user confirms before any spec is written.
- AC5: The spec `plan-it` writes has **Problem**, **Acceptance criteria**, **Out of scope**, **Decisions**, optional **Testing**, a `Spec version: N` line and a **Changelog**. Each criterion has a stable ID (`AC1`, `AC2`, …) that is never reused or renumbered across versions; behaviour criteria use EARS form (`When <trigger>, the <system> shall <response>`); other criteria are checkable statements. There are no user stories.
- AC6: When work does not fit one reviewable PR, `plan-it` splits it into child specs that each ship on their own, each a full spec with its own criteria IDs and a `Blocked by:` line, and stores none until the user approves the breakdown. The parent lists its children in dependency order.
- AC7: `build-it` carries the built-in coding method (list every place the change reaches first; take the first option that fully works: not needed → reuse → standard library or platform → installed dependency → least new code; no speculative options or abstractions; fix bugs at the root in shared code; never cut validation at trust boundaries, data-loss handling, security or accessibility; mark known limits with `shortcut: <limit>, <when to upgrade>`) and the built-in test method (test at the spec's Testing seam through public interfaces; red before green; no implementation-coupled or tautological tests). New non-trivial logic and bug fixes get a test that fails without the change, unless Out of scope excludes tests. Its hand-off lists what it skipped or did not check and any risk.
- AC8: When no house PR template exists, `stage-it` writes the body as Review focus, Summary, Evidence, How to verify, Merge danger, Follow-ups and Spec. Review focus names 1–3 places that need human judgment, with file links and a reason, or says "None". Visuals are text (markdown, Mermaid, `diff`), mark what changed (diff markers, Mermaid `classDef changed`, `# ← why` notes), and sit next to the text they support. The Spec block checks each criterion by ID against the diff (`- [x] AC1: …`, `- [ ] AC3: … — not met: <why>`).
- AC9: `review-it` reviews in a fresh-context subagent where the agent supports one, on four axes: **Spec** (criteria missing or partial by ID, unrequested behaviour, criteria done wrong), **Standards** (the repo's documented standards, citing file and rule), **Security** (OWASP Top 10 categories, each finding citing a CWE ID and a concrete exploit case) and **Compliance** (`## Rules`, `SECURITY.md`, policy docs, PR-template checklists, `CODEOWNERS` on sensitive paths, each finding citing its rule). With no documented compliance rules it reports `Compliance: no documented rules found` and invents no framework.
- AC10: Every `review-it` finding needs a concrete failing case, carries its axis tag and has four parts: what this is, the problem, the smallest fix, and what happens if it is skipped. Findings sort into one numbered Must fix / Should fix / Nice to have list. Broken rules, red gates, unmet criteria, exploitable security findings and compliance findings are always Must fix. The report opens with `What this change does:` and ends with `Verdict:` and `Not checked:`. review-it never edits code and is the only skill that marks a PR ready.
- AC11: `explore-it` keeps everything the `explorer` skill does (hard rules on secrets, scan layers, opt-in remote layer, 21 sections, review with the user, `ENV_MAP.md` user and repo maps, merge on re-run), uses "Source control / code host" for section 1, and after the map review proposes the phase files from confirmed patterns, one phase at a time, writing only what the user confirms to the layer the user picks. It never commits or pushes.
- AC12: The `/create-it` driver runs plan-it → build-it → stage-it → review-it with the fix loop, and the user takes part only in the interview. When plan-it splits work, the driver builds the first child with no blockers. When no config exists, it runs on defaults and says once `no create-it config; using defaults (run /explore-it to tailor)`.
- AC13: Each `SKILL.md` has a `## Tone` line: agent chat (status, progress, hand-off, loop verdicts) in caveman full; SKILL.md prose, specs, PR bodies, review findings, `ENV_MAP.md` and config docs in caveman lite (no filler or hedging, full sentences); interview questions, security warnings, irreversible actions and clarifications in normal prose.
- AC14: Every skill folder has a `CREDITS.md`, and the plugin root has one. Credits name external work only, as "inspired by" with what was taken: `grilling`, `to-spec`, `to-tickets`, `tdd`, `pr`, `code-review` (Matt Pocock, `mattpocock/skills`); `ponytail`, `ponytail-review` (Dietrich Gebert, `DietrichGebert/ponytail`); `show-me` (Dex Horthy, HumanLayer); `caveman` (Julius Brussee, `JuliusBrussee/caveman`). The same credits appear in each `SKILL.md` frontmatter as `metadata.credits.inspired_by` entries of `{skill, author, source, took}`. The root `CREDITS.md` also lists the standards used: EARS, OWASP Top 10, CWE.
- AC15: `docs/config.md` documents the three headings plus `## Code Host` (defined on first use as the platform that hosts the repo, pull requests and issues), the layering, and a solo and an enterprise example, with no `## Use`. The README and changelog say that `## Use` and `configure-it` are gone. No skill text uses "grill" or "grilling" except in credits.
- AC16: No skill ever merges or approves a PR, whatever the config says. `build-it` and `stage-it` never hand off on a red gate.

## Out of scope

- An eval suite or automated tests for the skills.
- A way to swap out a built-in method; a team adds extra checks as prose under `## Rules`.
- Bundling or vendoring other people's skills.
- Merging PRs.
- Writing glossary entries or ADRs during the interview.
- A fallback that reads old `## Use` lines or `## Forge` headings.
- Moving the toolkit's authored skills (`create-pr`, `grilling`, `eow-ship-report`) into `willdah/skills`.
- Migrating the toolkit (closing toolkit PR #9, installing create-it, moving the Forgejo `gh`→`tea` table and worktree rule into `~/.config/create-it/`) is follow-up work in the toolkit repo.

## Testing

No automated tests (see Out of scope). Each criterion is checked by reading the changed files: `grep` for `## Use`, `## Forge`, `grill`, `configure-it` and `skills/explorer` returns only changelog and credit lines.

## Decisions

- **Methods are built in, not plugged in.** The parts of other skills that proved effective are rewritten into each create-it skill. Trade-off: a team cannot swap a method, only add rules; consistent results matter more than swap-ability.
- **`## Use` is removed entirely.** An extra team check (for example a SOX review skill) goes under `## Rules` as prose; skills already read Rules as instructions.
- **Specs use EARS with stable IDs; no user stories.** EARS is short, checkable and unambiguous for agents and humans. IDs let build-it, the PR checklist and review findings cite `AC3` across versions. User stories add length without being checkable against a diff.
- **The interview reads domain docs generically.** No fixed glossary file name; the agent finds what the repo has.
- **Split children ship on their own** with blocking edges and user approval. No expand–contract rule.
- **The PR body leads with Review focus.** Text visuals with change emphasis serve the human reviewer and an agent reviewer alike; no HTML files, since a PR body cannot host them.
- **Review runs in a fresh-context subagent.** In a chained session the reviewer would otherwise share the author's context and bias. One subagent, not one per axis, to keep cost down.
- **Security is built in; compliance comes from the repo.** Security categories are generic (OWASP Top 10, CWE). Compliance frameworks differ per organisation, so review-it uses only documented rules. Scanners the repo already has run as Gates; review-it adds no tools.
- **explore-it replaces configure-it.** One skill learns the environment (`ENV_MAP.md`) and turns confirmed patterns into create-it's phase files. The map stays useful outside create-it through the pointer line in agent instructions.
- **`## Code Host`, not `## Forge` or "Version Control System".** The section maps PR and issue commands for the hosting platform; git is the version control system and is called directly.
- **The driver does not require explore-it.** A long interactive scan in front of a one-shot command costs more than running on defaults.
- **Tone follows the reader.** Caveman full for agent chat saves output tokens; lite for skill prose and human-read artifacts, because EARS needs grammar and reviewers need sentences.
- **Credits are "inspired by".** Nothing is called at runtime, so "optional default, not bundled" no longer fits. Only external work is credited; Will's own skills (`create-pr`, `explorer`) are not.
- **Version 0.3.0, breaking.** Removing `## Use`, `## Forge` and `configure-it` breaks existing config, with no fallback.
- Kept from earlier versions: config is prose, not YAML; Rules and Gates add up across layers; specs version themselves and only plan-it edits them; review reports and build fixes; PRs stay draft until review-it or a human says ready; the failing-test rule is fixed; shared rules repeat briefly in each `SKILL.md`; one marketplace repo for all of Will's plugins.

## Layout

```text
skills/                                  # repo: willdah/skills
├── .claude-plugin/marketplace.json      # lists create-it
├── plugins/create-it/
│   ├── .claude-plugin/plugin.json       # version 0.3.0
│   ├── skills/{create-it,explore-it,plan-it,build-it,stage-it,review-it}/{SKILL.md,CREDITS.md}
│   ├── docs/config.md
│   ├── docs/specs/create-it.md
│   ├── README.md
│   └── CREDITS.md
├── README.md
└── LICENSE                              # MIT
```

## Changelog

- v8 (2026-11-06): methods built in, `## Use` removed; EARS criteria with stable IDs and a Testing section; "interview" replaces "grill"; PR body adds Review focus, How to verify and Follow-ups; review-it adds Standards, Security and Compliance axes in a fresh-context subagent; explore-it (from `explorer`) replaces configure-it; `## Forge` renamed `## Code Host`; caveman tone; credits as "inspired by"; version 0.3.0.
- v7 (2026-11-02): renamed the plugin and its config paths to `create-it`, version 0.2.0; added the `create-it` driver skill, replacing the "no driver skill" decision and the umbrella-skill out-of-scope line.
- v6 (2026-11-02): renamed the plugin and its config paths to `do-it` (`docs/do-it/`, `~/.config/do-it/`), with no fallback to the old paths.
- v5 (2026-11-02): build-it's review-fix step may call review-it directly; the root `CREDITS.md` is at the plugin root, as in Layout.
- v4 (2026-11-02): credits list only external skills and their owners, not the plugin's own author.
- v3 (2026-11-02): config paths renamed to `docs/compose-it/<phase>.md` and `~/.config/compose-it/<phase>.md`, after the plugin.
- v2 (2026-11-02): housed in the `willdah/skills` marketplace repo under `plugins/compose-it/`, not its own repo.
- v1 (2026-11-02): first version, from the grilling session.
