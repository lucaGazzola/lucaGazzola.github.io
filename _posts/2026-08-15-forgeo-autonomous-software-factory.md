---
layout: post
title: "Forgeo: an autonomous software factory for coding agents"
date: 2026-08-15
description: How Forgeo turns a plain backlog into an application by running your coding agent autonomously.
---

AI coding agents are excellent at the work we hand them: implement a feature, fix a bug, refactor a module. What scales badly is everything *around* that work: deciding what to work on next, feeding tasks one at a time, integrating the results, and babysitting the run until it succeeds. [Forgeo](https://github.com/lucaGazzola/forgeo) is a project I have been building to automate that orchestration: a scheduled, agent-driven software factory that turns a backlog into merged commits, with a single human in the loop.

## A factory, not a chat session

The mental model is a factory line, not a chat window. A small daemon wakes up on a fixed interval and runs one production cycle: it picks the oldest *OPEN* task, runs your coding agent on it, and commits the result. The factory runs continuously; you are the foreman who stocks the backlog and handles the exceptions.

Forgeo is agent-agnostic: the `agent_command` in the configuration can be any CLI agent that can work in the repository. The task is handed over through the `FORGEO_TASK` environment variable, the exit code is the contract, and everything else is the agent's business. If you have a command line for an agent, Forgeo has a job for it.

## One cycle, end to end

Every cycle is deterministic and small:

1. **Pick a task.** The oldest `OPEN` task whose dependencies are all `COMPLETED`.
2. **Run the agent.** The repository is the working directory; the task text arrives via `FORGEO_TASK`. A timeout guards runaway runs.
3. **Integrate the result.** On success, Forgeo commits and pushes the work directly.
4. **Handle the outcome.** A `BLOCKED` exit code means the agent committed partial work and left you a decision: the task is marked blocked with the agent's reason attached. Any other failure discards the changes and marks the task `FAILED`.
5. **Idle time is productive.** When the backlog is empty, the same agent runs a refactoring pass over the codebase instead of doing nothing.

There is a per-repository run lock, so agent cycles never overlap.

## Blockers: the one place humans matter

The design goal is that Forgeo only interrupts you when a decision is genuinely yours. When the agent hits something that needs judgment it exits with the blocked code, and Forgeo materializes the problem as a single file: `BLOCKER.md`, a derived view of every blocked task with the agent's real reasons attached. You read it, decide, reopen the task, and the file disappears. Everything else flows through the pipeline without you.

## Built to be boring

A factory that runs unattended earns its keep by being reliable. The boring details are where Forgeo spends most of its effort:

- **Retries.** Transient failures (a network blip, a flaky test) are retried automatically when the retry policy is enabled, and only a task that keeps failing ever reaches you.
- **Backup discipline.** A file backlog is snapshotted before every agent run and on daemon startup, and restored automatically if it is ever found corrupt. A bad write never loses your tasks.
- **Fail-fast validation.** `forgeo validate` is a read-only dry run that checks the configuration, the git repo, the agent command, and the backlog before the first cycle ever runs.
- **Hot reload.** The daemon re-reads `forgeo.yaml` on the next cycle boundary when it changes (or on `SIGHUP`), so tuning the interval or the prompt does not require a restart.

## Observability without the infrastructure

The factory's state is stored in plain files you can open at any time: the backlog, a rotating log, the daemon state with the last outcome and next run. On top of that there are two layers of tooling:

- **The CLI**: `forgeo status` for a one-glance summary, `forgeo once` to run a single cycle in the foreground, `forgeo run --task <id>` to triage one specific task now.
- **The web console**: a central dashboard, `forgeo web`, that aggregates the backlog, run history, and logs of every instance on the host.

Multiple repositories are first-class: run several factories at once, one per repo, each with its own backlog, logs, and locks, and manage them all from the same CLI and dashboard.

## Try it

The whole loop is four commands:

```bash
curl -fsSL https://forgeo.org/install.sh | bash   # install
forgeo init                                        # wizard: config + backlog
forgeo start                                       # daemon, detached in background
forgeo status                                      # what the factory is doing
```

Fill the backlog, point `agent_command` at the agent CLI you already use, and watch the factory run. The source is on [GitHub](https://github.com/lucaGazzola/forgeo).
