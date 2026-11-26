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
| `/create-it` | Runs the whole loop as an orchestrator: plans with you, then on its own runs build, stage, each review round and each fix round in a new agent session, so every review starts unbiased. You only take part in planning. |
| `/explore-it` | Maps your environment (code host, tracker, CI, registries, cloud auth, conventions, gates) into `ENV_MAP.md` without recording secrets, then drafts the config below from what you confirm. |
| `/plan-it` | Confirms the goal, interviews you one question at a time, confirms functional and non-functional requirements, proposes up to 3 approaches, then writes a versioned spec for the smallest version that solves the problem: requirements with nested EARS acceptance criteria (`F1.1`…), out of scope, future follow-ups, testing, decisions. |
| `/build-it` | Builds the spec on a Conventional Branch with the smallest complete change and test-first logic, until the criteria and the definition of done are met and the gates pass. Also fixes review findings. |
| `/stage-it` | Runs the gates, pushes, and opens or updates a draft PR that leads with what needs your judgment, shows the change visually, and checks each criterion by ID against the diff. |
| `/review-it` | Reviews in a fresh context on four axes: spec, standards, security (OWASP, CWE), compliance. Reports only, never edits code. The only skill that marks a PR ready. |

Each phase skill also works alone, for example `/review-it #12` on a colleague's PR. The spec is what they hand to each other: `/build-it #42`.

## Configure

Each phase reads `docs/create-it/<phase>.md` in the repo, then `~/.config/create-it/<phase>.md`, then its defaults. Optional headings: `## Rules`, `## Gates`, `## Then` (what to chain into), and `## Code Host` in `stage.md`. Your personal rules add to the team's and cannot drop them. See [docs/config.md](docs/config.md) for the details and a solo and an enterprise example.

## Upgrading from 0.2

0.5 makes `/create-it` an orchestrator: each phase after planning runs in its own agent session and ends with a `create-it-report` block. Agents without subagents run the phases inline, with a warning. Phase skills called directly are unchanged.

0.4 makes specs iterative: plan-it confirms the goal and the functional and non-functional requirements, proposes up to 3 approaches, and writes acceptance criteria nested under each requirement (`F1.1`), with Future follow-ups; build-it and review-it share one definition of done. Older specs with flat `AC1` criteria still work.

0.3 builds the methods in. `## Use` is gone (write an extra team check as a Rules item), `## Forge` is now `## Code Host`, and `/explore-it` replaces `/configure-it`. There is no fallback for the old headings.

## Never

No skill merges or approves a PR, and only review-it marks one ready. review-it never edits code. build-it and stage-it never hand off on a red gate.

## Credits

The methods are rewritten from skills by Matt Pocock, Dietrich Gebert, Dex Horthy (HumanLayer) and Julius Brussee. See [CREDITS.md](CREDITS.md).
