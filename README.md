# willdah/skills

Plugins and skills for Claude Code and GitHub Copilot CLI, by Will ([willdah](https://github.com/willdah)).

Claude Code:

```
/plugin marketplace add willdah/skills
/plugin install compose-it@willdah-skills
```

GitHub Copilot CLI:

```
copilot plugin marketplace add willdah/skills
copilot plugin install compose-it@willdah-skills
```

| Plugin | What it does |
| --- | --- |
| [compose-it](plugins/compose-it) | SDLC skills (plan, build, stage, review) that follow your environment's config instead of hard-coding which skills to use. |
| [explorer](plugins/explorer) | Maps your SDLC environment (forge, tracker, registries, cloud auth, conventions, gates) into `ENV_MAP.md` for you and your agents, without recording secrets. |

MIT licensed. Each plugin's `CREDITS.md` names the people whose skills and ideas it builds on.
