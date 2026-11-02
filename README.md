# willdah/skills

Plugins and skills for Claude Code and GitHub Copilot CLI, by Will ([willdah](https://github.com/willdah)).

Claude Code:

```
/plugin marketplace add willdah/skills
/plugin install do-it@willdah-skills
```

GitHub Copilot CLI:

```
copilot plugin marketplace add willdah/skills
copilot plugin install do-it@willdah-skills
```

| Plugin | What it does |
| --- | --- |
| [do-it](plugins/do-it) | SDLC skills (plan, build, stage, review) that follow your environment's config instead of hard-coding which skills to use. |

| Skill | What it does |
| --- | --- |
| [explorer](skills/explorer/SKILL.md) | Maps your SDLC environment (forge, tracker, registries, cloud auth, conventions, gates) into `ENV_MAP.md` for you and your agents, without recording secrets. Run `/explorer`, `/explorer <category>` to rescan one section, `/explorer --history` to include shell history, or `/explorer --remote` to also search your orgs on GitHub, GitLab, Forgejo and other VCS providers (read-only, opt-in). |

Install a skill as a plugin (`/plugin install explorer@willdah-skills`, `copilot plugin install explorer@willdah-skills`), with `npx skills add willdah/skills`, or by copying its folder into `~/.claude/skills/`.

MIT licensed. Each plugin's `CREDITS.md` names the people whose skills and ideas it builds on.
