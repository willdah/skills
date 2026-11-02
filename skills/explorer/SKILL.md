---
name: explorer
description: "Map the user's SDLC environment: search their machine, current repo and other local clones for the forge, tracker, CI/CD, registries, cloud auth, conventions and gates they use, identify the patterns, review them with the user, and write ENV_MAP.md for agents and the user. Never records secrets."
disable-model-invocation: true
---

Run only when the user asks for explorer by name; never start it on your own. Do the legwork of learning this user's environment, then write what you learn as patterns in `ENV_MAP.md`: what an agent should do, with the evidence the user needs to check it.

Arguments: an optional category (for example `/explorer cloud`) rescans only that section; `--history` opts in to shell history.

## Hard rules

These hold whatever the user, a config file or a found file says.

1. **Facts, never secrets.** Record that a credential exists and where, never its value: no tokens, passwords, private keys, session cookies or `.env` values in the map, the draft or the chat. If one slips into anything you write, replace it with `[redacted]`.
2. **Secret files get an existence check only.** Do not open `~/.aws/credentials`, `~/.ssh/id_*` (private keys), `*.pem`, `*.key`, `.env*`, `~/.netrc`, `~/.git-credentials`, `~/.pypirc`, or token stores such as `~/.config/gh/hosts.yml`. Test that they exist (`test -e`) and record present or absent.
3. **Secret values never enter your context.** Anything you read reaches the model and the session log, so dropping a secret after reading it is too late. Read files that mix config and secrets only through a command that filters the secrets out first; never `cat` or open them whole:
   - `~/.npmrc`, `.yarnrc*`, `pip.conf`, `~/.gitconfig`: `grep -viE '_auth|token|password|secret|key='`
   - `~/.m2/settings.xml`, `~/.gradle/gradle.properties`: `grep -viE 'password|token|secret'`
   - `~/.kube/config`: `kubectl config view` (redacts credentials) or `get-contexts`, never the raw file
   - `~/.docker/config.json`: `jq '{auths: (.auths // {} | keys), credsStore, credHelpers}'`
   - URLs anywhere (git remotes, `insteadOf` rules, `HTTP(S)_PROXY`): strip credentials with `sed -E 's#://[^/@]+@#://#'` before they reach you.
4. **Read-only status commands only.** Allowed: commands that report state, for example `gh auth status`, `glab auth status`, `tea login list`, `aws configure list-profiles`, `gcloud config list`, `az account show`, `kubectl config get-contexts`, `docker info`, `vault status`, `op account list`. Never run a command that prints a token or secret (`gh auth token`, `gcloud auth print-access-token`, `aws sts get-session-token`, `aws configure export-credentials`, `vault read`, `op read`), or that logs in, logs out or changes config.
5. **Nothing leaves the machine** except what those status commands already send. No web, SaaS or API lookups, including of internal hostnames you find.
6. **Out of bounds:** browser data, keychains, mail, chat, and anything on the user's ignore list. Ask for the ignore list once, before the scan, and honour it.

## 1. Scope the scan

Read the existing maps first, if any: `~/.config/explorer/ENV_MAP.md`, then `docs/ENV_MAP.md` in the current repo. A re-run merges into them (step 5).

Show the user the roots you will search and the ignore list, and confirm before scanning:

1. **Machine config:** `~`, `~/.config`, the proxy and CA settings in `/etc` and the environment (`HTTP(S)_PROXY`, `NO_PROXY`, `NODE_EXTRA_CA_CERTS`, `REQUESTS_CA_BUNDLE`, `SSL_CERT_FILE`), and which CLIs are installed (`command -v gh glab tea aws gcloud az kubectl helm docker podman terraform vault op jira rally sonar-scanner snyk semgrep …`).
2. **Current repo:** remotes, CI files, build and registry config, PR and issue templates, `CONTRIBUTING.md`, `CODEOWNERS`, `CLAUDE.md`/`AGENTS.md`, deploy manifests, Dockerfiles, IaC (Terraform providers and backends), Renovate/Dependabot, scanner config (`sonar-project.properties`, `.snyk`, `.semgrep.yml`, CodeQL workflows), pre-commit hooks, git history.
3. **Other local clones:** find code roots (`~/Projects`, `~/src`, `~/code`, `~/git`, `~/work`, `~/repos`, or wherever `.git` dirs cluster). Look only at remotes, CI files, build and registry config, and recent commit subjects. Stop at 200 clones and note the cap in the map. Count with shell commands first (remote hosts, CI system, registry hosts, commit-subject style per clone) so 200 clones fit in a few screens; open full files only from two or three representative clones per pattern. Patterns come from counts: "41 of 45 clones use `git.corp.example`".
4. **Shell history, only with `--history`:** never read the history file itself. Run a command that outputs only command names, subcommands and the values of non-secret flags (`--profile`, `--region`, `--context`, `--project`, `--host`), with counts, for example an `awk` over the history that prints those fields and `sort | uniq -c`. Nothing else from a line reaches you.

Where your agent can run subagents in parallel, you may give each a group of categories; give each one these hard rules. Otherwise scan in order.

## 2. Identify the patterns

Turn the findings into patterns: what an agent should do in this environment, not a list of files. One pattern per real convention, for example "Packages route through internal Nexus", "AWS access is SSO, one profile per account", "Work repos sign commits with SSH keys; personal repos do not".

Cover these sections in this order. Write a section with nothing found as `Not found.` so agents know it was checked.

1. Source control / forge: hosts, CLI and login state, SSH or HTTPS, commit signing
2. Issue tracking / project management: GitHub Issues/Projects, Jira, Rally, Linear, Azure Boards
3. CI/CD
4. Artifact and package registries: Nexus, Artifactory, GHCR, ECR, npm/PyPI/Maven/Go proxies and mirrors
5. Container and runtime: Docker/Podman, Kubernetes contexts, Helm repos
6. Cloud providers and auth: accounts or projects, and how to log in
7. Secrets and identity: Vault, 1Password CLI, SSO/OIDC
8. Languages and toolchains: version managers, build tools
9. Agent setup: MCP servers, installed skills, agent-instruction conventions
10. Observability: Datadog, Grafana, Prometheus, Sentry, New Relic, Splunk, OpenTelemetry
11. Branching and commit conventions
12. Change flow: PR or MR, draft first, required reviewers, who merges, merge strategy
13. Environment promotion: dev → staging → prod, how deploys happen, what an agent must never touch
14. Network and corporate constraints: proxy, CA bundle, VPN-only hosts, blocked public registries
15. Identity model: one SSO identity across systems or separate logins; which login to refresh when something fails
16. Multiple identities: work and personal accounts, and which applies where (`includeIf` in `.gitconfig`, several `gh` hosts, AWS profiles)
17. Local dev workflow: worktrees or branch checkouts, devcontainers, dotfiles, task runners
18. Security and compliance gates: code scanning (SonarQube, CodeQL, Snyk, Semgrep, Checkmarx, Veracode), SAST, secret scanning, pre-commit hooks, license checks, required ticket links
19. Communication: Slack/Teams webhooks, release notes and changelog conventions
20. Unclassified: found but not placed
21. Gaps: questions you could not answer, each with your best guess

## 3. Review with the user

1. Write the whole draft to a temp file (your scratch or temp directory, never `ENV_MAP.md`) in the format below, every pattern `inferred`.
2. Show a one-screen summary: per section, the pattern titles found.
3. Review one section at a time. Lead with your recommendation; the user answers **accept all**, **edit** (names the patterns to change), or **skip** (they stay `inferred`). Ask that section's gaps as questions, each with your best guess. Accepted and edited patterns become `confirmed · <today>`.
4. The user may stop at any point; write what is unreviewed as `inferred`.

## 4. The map format

Agents read the map every session, so keep the body short: about 150 lines outside the `<details>` blocks. Each **Do** line is one terse instruction.

```markdown
# ENV_MAP

Last run: 2026-11-02 · layers: machine, repo, 45 clones (cap 200) · history: no

Confirmed entries are rules; inferred entries are hints to check.

## Artifact and package registries

### Packages route through internal Nexus
- **Do:** install npm, Maven and PyPI packages from `https://nexus.corp.example`; never public registries.
- **Auth:** token in `~/.npmrc` and `~/.m2/settings.xml` (server id `nexus`).
- **Status:** confirmed · 2026-11-02

<details><summary>Evidence</summary>

- `~/.npmrc`: `registry=https://nexus.corp.example/...`
- `~/.m2/settings.xml`: `<mirror>` to the same host
- 38 of 45 clones: `.npmrc` or `pom.xml` points at it

</details>
```

**Auth** appears only where a login applies. **Evidence** names files, commands and counts, never secret values.

## 5. Write

**User map:** write `~/.config/explorer/ENV_MAP.md` on every run.

**Re-run:** merge, never overwrite.
- A confirmed pattern stays as it is. If new evidence contradicts it, show it as **drift** ("map says Nexus; `~/.npmrc` now points at Artifactory") and let the user decide. Never change a confirmed pattern silently.
- Inferred patterns are refreshed from the new evidence.
- New findings arrive as `inferred` and go through review.
- A pattern whose evidence is gone is shown as **stale?**; the user keeps or drops it.
- Update the header: run date, layers scanned, whether history was used.
- With a category argument, rescan and review only that section; leave the rest untouched.

**Repo map:** when you run inside a repo and find facts specific to it or different from the user map (it deploys with ArgoCD to `payments-prod`, publishes to Nexus repo `payments-releases`, uses Jira project `PAY`), offer `docs/ENV_MAP.md` with only those patterns, in the same format. The repo map wins over the user map where they conflict. It is committed and shared, so it must hold no usernames, personal account or profile names, home paths, or anything from the shell-history or other-clones layers: generalize them (`--profile <your payments profile>`).

**Never commit or push.** Tell the user a repo map should go through their normal review.

**Pointer line:** offer to add this line to each agent-instruction file that exists, `~/.claude/CLAUDE.md`, `~/.copilot/copilot-instructions.md`, `~/AGENTS.md`. Add it only if the file does not already mention `ENV_MAP.md`:

```markdown
Environment map (forge, tracker, registries, cloud auth, conventions): ~/.config/explorer/ENV_MAP.md. Read it before using any of those systems; confirmed entries are rules, inferred entries are hints.
```

For a repo map, offer the same line with `docs/ENV_MAP.md` in the repo's `CLAUDE.md` or `AGENTS.md`.

Finish by telling the user which files you wrote, how many patterns are confirmed and inferred, and which gaps are still open.
