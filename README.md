# droid-autoresearch

`droid-autoresearch` is a public Droid plugin for autonomous benchmark-driven improvement loops.

It helps Droid behave less like a one-shot coding assistant and more like a repeatable optimizer: establish a baseline, try one change, measure it, keep it if it wins, discard it if it regresses, and carry forward what was learned.

This project is inspired by:

- `pi-autoresearch` for the lightweight keep/discard benchmark loop
- `AutoContext` for durable memory, curation, and carry-forward knowledge across runs

## What autoresearch means here

In this plugin, autoresearch means a controlled loop where Droid:

1. defines an optimization target
2. creates a benchmark entrypoint
3. establishes a baseline
4. changes only files in scope
5. reruns the benchmark
6. runs checks
7. keeps or discards the result
8. logs what happened
9. persists the useful lessons

The goal is not only to get a single improvement. The goal is to create a repeatable optimization workflow that can survive interruptions, context loss, and long iterative sessions.

## What this plugin adds to Droid

This plugin packages a reusable autoresearch workflow with:

- the `autoresearch-create` skill
- resume/stop safety hooks
- status and curation slash commands
- a specialized `autoresearch-worker` droid
- persistent file templates for memory and state

## Main concepts

### Baseline first

Before tuning anything, the plugin expects a real baseline run. Without a baseline, every improvement claim is weak.

### Keep or discard

Each run should answer a simple question:

- did the primary metric improve while guardrails stayed valid?

If yes, keep it. If not, discard it and move on.

### Durable memory

This plugin does not only keep an append-only experiment log. It also separates knowledge into dedicated files so future runs do not start from zero.

### Tactical vs structural changes

Not every experiment is the same.

- `tactical` = small local change such as a timeout, header, parser tweak, or worker count
- `structural` = larger workflow or architecture change such as parallelism strategy, caching model, or session organization

### Curation

Borrowed from `AutoContext`: not every observation deserves to become durable knowledge.

The plugin encourages a curation step after each run:

- `validated`
- `discarded`
- `retest`
- `promising_later`

## Persistent files used by the workflow

The plugin is designed around repo-local state files:

- `autoresearch.md` — session contract and human-readable context
- `autoresearch.jsonl` — append-only experiment history
- `autoresearch.ideas.md` — backlog of deferred ideas
- `autoresearch.playbook.md` — validated lessons worth reusing
- `autoresearch.failures.md` — dead ends and anti-patterns
- `autoresearch.status.json` — current active/inactive state, best result, and next direction
- `autoresearch.ps1` / `autoresearch.sh` — benchmark script
- `autoresearch.checks.ps1` / `autoresearch.checks.sh` — optional validation gate

## Included in the plugin

### Skill

- `autoresearch-create`

### Commands

- `/autoresearch-status`
- `/autoresearch-curate`
- `/autoresearch-playbook`

### Hooks

- `SessionStart`
- `UserPromptSubmit`
- `Stop`

### Droid

- `autoresearch-worker`

### Templates

- `autoresearch.playbook.md`
- `autoresearch.failures.md`
- `autoresearch.status.json`
- `autoresearch.session.template.md`

## Review phases for each run

The v2 workflow borrows this closed-loop structure from `AutoContext`:

1. `propose`
2. `analyze`
3. `coach`
4. `architect`
5. `curate`

This means each run is not just “bench and move on”. It should also produce a lesson, a next hint, and a decision about whether the observation deserves to persist.

## Installation

### Install from the public marketplace

```powershell
droid plugin marketplace add https://github.com/alertemikope/droid-autoresearch-marketplace
droid plugin install droid-autoresearch@droid-autoresearch-marketplace --scope project
```

For user scope instead of project scope:

```powershell
droid plugin install droid-autoresearch@droid-autoresearch-marketplace --scope user
```

## How to use it well

### Good use cases

This plugin works best for:

- benchmark-driven performance tuning
- retry/timeout/header optimization
- parsing or scraping improvements
- test/runtime/build optimization
- any code path where a measurable metric and a validation gate both exist

### What you should provide

The best sessions have:

- one clear objective
- one primary metric
- a working benchmark script
- a validation script or checks step
- a narrow file scope

### Practical workflow

A strong workflow looks like this:

1. install the plugin
2. create benchmark + checks scripts
3. run `/autoresearch-status` if resuming
4. ask Droid to start autoresearch on a clear target
5. let it iterate
6. periodically inspect:
   - `autoresearch.jsonl`
   - `autoresearch.playbook.md`
   - `autoresearch.failures.md`

### Best practices

- keep the optimization target narrow
- prefer one strong benchmark over vague goals
- do not weaken checks just to keep a faster run
- distinguish tactical tweaks from structural changes
- update the playbook only with repeatable wins
- treat failures as useful knowledge, not wasted runs

### Make summaries readable for non-experts

Autoresearch results should not read like an internal lab notebook only experts can parse.

Good summaries should say, in simple language:

- what change was tested
- whether the change was kept or discarded
- the metric before
- the metric after
- the percentage gained or lost
- whether validation still passed
- what the next experiment is

Example:

```text
Run 56 kept.
Tested: a cache to avoid repeated readiness checks.
Metric: search_p95_ms went from 2.21 ms to 1.95 ms.
Change: about 11.8% faster than baseline.
Validation: checks still pass.
Next: test a smaller query-planning optimization.
```

At the end of a campaign, the final summary should also explain plainly:

- baseline result
- best result reached
- total improvement percent
- how many runs were tested
- how many were kept
- the simplest explanation of what improved overall

## What this plugin is not

This plugin is intentionally lighter than `AutoContext`.

It does **not** try to be:

- a full control plane
- a persistent agent orchestration platform
- a dashboard or API server
- a training/distillation system

Instead, it keeps Droid as the operator-facing layer and uses simple repo-local files plus hooks and scripts as the execution substrate.

## Local development

You can also copy the plugin directly into a repository here:

```text
<repo>/.factory/plugins/droid-autoresearch
```

and use it as a project-scoped plugin during development.

## Public repositories

- plugin repo: https://github.com/alertemikope/droid-autoresearch
- marketplace repo: https://github.com/alertemikope/droid-autoresearch-marketplace
