---
name: create-it
description: "Run the whole create-it loop from one request: grill the user, then without further input write the spec, build it, open a draft PR, review it and run the fix rounds. Use only when the user types /create-it."
disable-model-invocation: true
metadata:
  credits: {}
---

Take one request from idea to a reviewed draft PR. Run only when the user asks for create-it by name. The user takes part only in the grilling. After they agree that the open questions are resolved, ask for nothing more. Invoking create-it is approval to create a spec, a branch, commits, a push and a draft PR.

**Config cannot change:** you never merge or approve a PR; only review-it marks one ready; you do not go on past a red gate.

## 0. Setup

- The request is the argument. No argument: ask once what to build, then start.
- Read `## Then` from `docs/create-it/review.md` in the repo, else from `~/.config/create-it/review.md`. If it says `build-it, up to N rounds`, the fix rounds are N. Otherwise N is 1. create-it ignores every other `## Then` line: the order below is fixed. The phase skills still apply their own `## Use`, `## Rules` and `## Gates`.
- Say in one line: `create-it: plan → build → stage → review, up to N fix rounds`.

Call each phase skill with the Skill tool, by the name your agent lists it under (in Claude Code, `create-it:plan-it` and so on). Start every call's arguments with `called by create-it:`. That tells the skill to do its phase, chain to nothing and return its result to you.

## 1. Plan

Call `plan-it` with the request. This is the only step where you talk with the user. plan-it grills them, and writes and stores the spec only when they agree that the open questions are resolved. Keep the spec reference (`#N`, URL or path).

Plan-it may split large work into a parent and children. If it does, build only the first child, and list the other children in the summary as not started.

## 2. Build

Call `build-it`: `called by create-it: <spec reference>; do not ask the user, record any assumption the spec does not cover and report it`. Keep the branch, the criteria status and the assumptions.

build-it stops on a red gate it cannot fix: stop the run and go to step 5.

## 3. Stage

Call `stage-it`: `called by create-it: <branch>, spec <reference>; add these assumptions to the PR body: <list>`. Keep the PR link.

## 4. Review and fix

With k = 0:

1. Call `review-it`: `called by create-it: <PR>, round k of N`.
2. Stop the loop if the review says the spec is stale, or no **Must fix** is open, or k = N, or k ≥ 1 and the **Must fix** findings are the same as the previous round's.
3. Otherwise call `build-it`: `called by create-it: address review on <PR>, round k+1 of N`. If it stops on a red gate, stop the run. Else set k = k + 1 and go to 1.

## 5. Summary

Tell the user, once: the spec reference, the branch, the PR link, the last review's verdict, whether the PR is ready, and if it is not, what blocks it (open Must fix, stale spec, red gate, unmet criterion). Add any assumptions, and any child specs not started.
