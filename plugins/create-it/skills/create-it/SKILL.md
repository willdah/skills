---
name: create-it
description: "Run the whole create-it loop from one request: plan it with the user (goal, interview, requirements, approach), then without further input write the spec, build it, open a draft PR, review it and run the fix rounds. Use only when the user types /create-it."
disable-model-invocation: true
metadata:
  credits:
    inspired_by:
      - skill: caveman
        author: Julius Brussee
        source: https://github.com/JuliusBrussee/caveman
        took: terse tone for agent chat
---

Take one request from idea to a reviewed draft PR. Run only when the user asks for create-it by name. The user takes part only in plan-it: confirming the goal, the interview, confirming the requirements and picking an approach. After plan-it stores the spec, ask for nothing more. Invoking create-it is approval to create a spec, a branch, commits, a push and a draft PR.

**Config cannot change:** you never merge or approve a PR; only review-it marks one ready; you do not go on past a red gate.

## Tone

Agent chat (status, progress, the summary): terse, fragments OK. Warnings and anything the user asks to clarify: normal prose.

## 0. Setup

- The request is the argument. No argument: ask once what to build, then start.
- Read `## Then` from `docs/create-it/review.md` in the repo, else from `~/.config/create-it/review.md`. If it says `build-it, up to N rounds`, the fix rounds are N. Otherwise N is 1. create-it ignores every other `## Then` line: the order below is fixed. The phase skills still apply their own `## Rules` and `## Gates`.
- Say in one line: `create-it: plan → build → stage → review, up to N fix rounds`.
- No create-it config at either level: run on defaults and say once `no create-it config; using defaults (run /explore-it to tailor)`. Do not require explore-it first.

## Delegate

You are the orchestrator. Plan with the user here; run every later phase in a new agent session that does not inherit this conversation (in Claude Code, the `Agent` tool, not a fork), so each phase starts with a fresh context and each review sees the change as a colleague would. Never continue an earlier session: each build, stage, review round and fix round gets its own.

Each delegated prompt says: run the phase skill, by the name your agent lists it under (in Claude Code, `create-it:build-it` and so on), with the arguments given below, which start `called by create-it:`; then end with the report below. Pass only what the phase needs: the spec reference, the branch or PR, the round, and for stage-it the assumptions. Paste the report schema into every prompt.

**Report.** Each phase ends its reply with one block like this. It fills the keys it owns and writes `none` for the rest:

````
```create-it-report
phase: <plan-it | build-it | stage-it | review-it>
status: <done | blocked>
blocked: <the reason, or none>
spec: <reference>
branch: <branch>
pr: <PR link>
criteria:
- <ID> <met | not met: why>
assumptions:
- <assumption>
verdict: <review verdict>
must_fix:
- <numbered Must fix finding, one line>
spec_stale: <yes | no>
ready: <yes | no>
```
````

Keep only these fields (and, from plan-it, the other child references). Do not read the diff, logs or findings yourself; that detail belongs to the phase sessions. Make every decision below from the report. Read only the last `create-it-report` block in a reply. A reply with no block, or one you cannot read, counts as `status: blocked`.

**No subagents.** If your agent cannot start a new session, run each phase inline with the skill tool (in Claude Code, `Skill`), with the same arguments. Say once `no subagents; running phases inline (review not isolated)`, and repeat it in the summary.

## 1. Plan

Call `plan-it` inline, with the skill tool: `called by create-it: <request>`. This is the only step where you talk with the user. plan-it confirms the goal, interviews them, confirms the requirements and has them pick an approach, and writes and stores the spec only when they agree. Keep `spec` from its report. If `status: blocked`, go to step 5.

Plan-it may split large work into a parent and children. If it does, its `spec` is the first child that nothing blocks: build only that one. Keep the other children's references from its reply, and list them in the summary as not started.

## 2. Build

In a new session, run `build-it`: `called by create-it: <spec reference>; do not ask the user, record any assumption the spec does not cover and report it`. Keep `branch`, `criteria` and `assumptions`.

`status: blocked` (a red gate it cannot fix): stop the run and go to step 5.

## 3. Stage

In a new session, run `stage-it`: `called by create-it: <branch>, spec <reference>; add these assumptions to the PR body: <list>`. Keep `pr`. If `status: blocked`, stop the run and go to step 5.

## 4. Review and fix

With k = 0:

1. In a new session, run `review-it`: `called by create-it: <PR>, spec <reference>, round k of N`. Pass nothing else: no assumptions, no earlier findings, no build reasoning.
2. Stop the loop if `status: blocked`, or `spec_stale: yes`, or `must_fix` is `none`, or k = N, or k ≥ 1 and `must_fix` is the same as the previous round's (the same file and problem, whatever the wording).
3. Otherwise, in a new session, run `build-it`: `called by create-it: address review on <PR>, round k+1 of N`. If `status: blocked`, stop the run. Else set k = k + 1 and go to 1.

## 5. Summary

Tell the user, once, from the reports: the spec reference, the branch, the PR link, the last review's verdict, whether the PR is ready, and if it is not, what blocks it (open Must fix, stale spec, red gate, unmet criterion, a blocked phase). Add any assumptions, any child specs not started, and the no-subagents warning if it applied.
