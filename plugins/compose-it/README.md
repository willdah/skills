# compose-it

Five skills that take work from requirement to a reviewed PR, and follow your environment's rules instead of hard-coding which skills to use.

```
/plugin marketplace add willdah/skills
/plugin install compose-it@willdah-skills
```

| Skill | Does |
| --- | --- |
| `/configure-it` | Reads your repo (CI, templates, CONTRIBUTING, CODEOWNERS) and drafts the config below. |
| `/plan-it` | Grills you on the requirements, then writes a versioned spec: problem, acceptance criteria, out of scope, decisions. |
| `/build-it` | Builds the spec on a Conventional Branch until the criteria are met and the gates pass. Also fixes review findings. |
| `/stage-it` | Runs the gates, pushes, and opens or updates a draft PR that shows the change and checks the criteria against the diff. |
| `/review-it` | Reviews against the spec and your rules. Reports only, never edits code. The only skill that marks a PR ready. |

Each skill works alone, for example `/review-it #12` on a colleague's PR. The spec is what they hand to each other: `/build-it #42`.

## Configure

Each phase reads `docs/compose-it/<phase>.md` in the repo, then `~/.config/compose-it/<phase>.md`, then its defaults. Four optional headings: `## Use` (which skills), `## Rules`, `## Gates`, `## Then` (what to chain into). Your personal rules add to the team's and cannot drop them. See [docs/config.md](docs/config.md) for the rules and a solo and an enterprise example.

## Defaults

| Phase | Default skill | Without it |
| --- | --- | --- |
| plan | `grilling`, `to-spec`, `to-tickets` | built-in interview and spec format |
| build | `ponytail:ponytail` (full), `tdd` | built-in smallest-change and test-first guidance |
| stage | none | built-in PR template |
| review | `ponytail:ponytail-review` | built-in review checklist |

None are bundled. Install the ones you want; see [CREDITS.md](CREDITS.md) for their owners.

## Never

No skill merges or approves a PR, and only review-it marks one ready. review-it never edits code. build-it and stage-it never hand off on a red gate.
