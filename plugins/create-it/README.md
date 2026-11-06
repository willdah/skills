# create-it

Six skills that take work from requirement to a reviewed PR, with proven methods built in and your environment's rules on top.

Claude Code:

```
/plugin marketplace add willdah/skills
/plugin install create-it@willdah-skills
```

GitHub Copilot CLI:

```
copilot plugin marketplace add willdah/skills
copilot plugin install create-it@willdah-skills
```

| Skill | Does |
| --- | --- |
| `/create-it` | Runs the whole loop: interviews you, then on its own writes the spec, builds it, opens a draft PR, reviews it and runs the fix rounds. You only answer the interview. |
| `/explore-it` | Maps your environment (code host, tracker, CI, registries, cloud auth, conventions, gates) into `ENV_MAP.md` without recording secrets, then drafts the config below from what you confirm. |
| `/plan-it` | Interviews you one question at a time, then writes a versioned spec: problem, EARS acceptance criteria with stable IDs (`AC1`…), out of scope, testing, decisions. |
| `/build-it` | Builds the spec on a Conventional Branch with the smallest complete change and test-first logic, until the criteria are met and the gates pass. Also fixes review findings. |
| `/stage-it` | Runs the gates, pushes, and opens or updates a draft PR that leads with what needs your judgment, shows the change visually, and checks each criterion by ID against the diff. |
| `/review-it` | Reviews in a fresh context on four axes: spec, standards, security (OWASP, CWE), compliance. Reports only, never edits code. The only skill that marks a PR ready. |

Each phase skill also works alone, for example `/review-it #12` on a colleague's PR. The spec is what they hand to each other: `/build-it #42`.

## Configure

Each phase reads `docs/create-it/<phase>.md` in the repo, then `~/.config/create-it/<phase>.md`, then its defaults. Optional headings: `## Rules`, `## Gates`, `## Then` (what to chain into), and `## Code Host` in `stage.md`. Your personal rules add to the team's and cannot drop them. See [docs/config.md](docs/config.md) for the details and a solo and an enterprise example.

## Upgrading from 0.2

0.3 builds the methods in. `## Use` is gone (write an extra team check as a Rules item), `## Forge` is now `## Code Host`, and `/explore-it` replaces `/configure-it`. There is no fallback for the old headings.

## Never

No skill merges or approves a PR, and only review-it marks one ready. review-it never edits code. build-it and stage-it never hand off on a red gate.

## Credits

The methods are rewritten from skills by Matt Pocock, Dietrich Gebert, Dex Horthy (HumanLayer) and Julius Brussee. See [CREDITS.md](CREDITS.md).
