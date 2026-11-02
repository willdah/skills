# explorer

One skill that does the legwork of learning your environment: which forge and tracker you use, whether packages go through an internal Nexus or Artifactory, which clouds you use and how you log in, and the conventions and gates your work follows. It writes what it finds as patterns in `ENV_MAP.md`, for you to review and for your agents to read.

Claude Code:

```
/plugin marketplace add willdah/skills
/plugin install explorer@willdah-skills
```

GitHub Copilot CLI:

```
copilot plugin marketplace add willdah/skills
copilot plugin install explorer@willdah-skills
```

## Use

| Command | Does |
| --- | --- |
| `/explorer` | Scans your machine config, the current repo and your other local clones, then reviews the patterns with you one section at a time. |
| `/explorer <category>` | Rescans one section, for example `/explorer cloud`. |
| `/explorer --history` | Also reads shell history: command names and non-secret flags only. |

It writes `~/.config/explorer/ENV_MAP.md`. Inside a repo with facts of its own, it offers a team-safe `docs/ENV_MAP.md` too, which wins where the two conflict. It then offers one pointer line for your `CLAUDE.md` or `AGENTS.md` so agents read the map.

Each pattern is `inferred` until you confirm it. Agents treat confirmed patterns as rules and inferred ones as hints. Running it again merges: confirmed patterns never change without asking you.

## What it never does

- Record a token, password, key, cookie or `.env` value. Secret files are only checked for existence.
- Run a command that prints a secret, logs in or changes config. It runs status checks such as `gh auth status`.
- Look anything up online, or read browser data, keychains, mail or chat.
- Commit or push.
