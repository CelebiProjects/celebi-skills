---
name: celebi
description: Explain the Celebi and Yuki workflow architecture and route analysis-project work to the relevant Celebi skill.
---

# Celebi and Yuki

Celebi manages analysis objects, dependencies, and reproducible snapshots called
impressions. Yuki is the execution and data service exposed through the DITE
connection. It coordinates REANA, native Snakemake, and SSH runners, logs, result
collection, and runner caches. A runner name such as `local` is a registered name;
it does not identify the backend by itself.

## Object model

- Project: root namespace and project identity.
- Directory: organizational object; directories **can be nested**.
- Algorithm: reusable code and execution commands.
- Task: execution parameters and inputs, with either a linked algorithm or its
  own non-empty `commands` in `celebi.yaml`.
- Data: a task flavor (`rawdata`, `datalist`, or `lhcb_ap_datalist`), not a fifth
  metadata object type.

Objects have `.celebi/config.json` and a README. Local settings may also live in
`.celebi/config.local.json`. Use Celebi commands for metadata and dependency
changes; edit source files and `celebi.yaml` normally. Do not manufacture object
metadata with ordinary `mkdir` or hand-written dependency JSON.

Task commands take precedence over linked algorithm commands. A linked algorithm
is optional for inline tasks; data tasks do not require one. Parameter substitution
uses the task's `parameters` mapping; do not assume algorithm defaults are merged.

## Choose a skill

| Work | Skill |
|------|-------|
| Explore objects, paths, or dependency edges | `celebi-navigation` |
| Create directories, algorithms, tasks, or data pointers | `celebi-object-creation` |
| Implement reusable computation and its command template | `celebi-algorithm-expert` |
| Configure task commands, parameters, inputs, and environment | `celebi-task-expert` |
| Submit and monitor the intended execution scope | `celebi-workflow-execution` |
| Register runners, resolve environments, or book REANA catalogs | `celebi-runner-management` |
| Register/verify data and transfer it between service locations | `celebi-data-management` |
| Inspect provenance, publish bookkeeping, migrate/archive snapshots, or clean retained data | `celebi-impression-management` |
| Deploy or diagnose the Flask/Celery service and its storage | `yuki-service-management` |
| Assess a slow or apparently stalled workflow | `celebi-workflow-patience` |
| Diagnose and repair a failure | `celebi-amend-expert` |
| Read logs, collect outputs, or inspect results | `celebi-output-checker` |

Load only skills needed for the work, using the environment's available skill
mechanism. If a companion skill is not installed, proceed with the available
instructions and CLI/source documentation; no particular `Skill` tool is required.

## Working conventions

Use the three entry points according to their scope: `celebi` enters the persistent
interactive shell, `celebi-cli COMMAND` runs one operation, and
`celebi-git COMMAND` contains Git integration. In the interactive shell, bare
`helpme` is object-aware; `help COMMAND`, `helpme COMMAND`, and `COMMAND --help`
show the same detailed command documentation.

Run standalone commands with the OS working directory set to the target object.
Use shell `cd` or the execution tool's working-directory option. `celebi-cli cd`
cannot change its parent shell's working directory.

```bash
celebi-cli tree
celebi-cli ls
celebi-cli predecessors
celebi-cli status
```

`tree` shows containment. `predecessors`, `successors`, and `draw-dag` expose
execution dependencies. `@/tasks/source` is a project-root path supported by task
linking commands; it is not an OS path to pass to shell `cd`.

For algorithm-backed tasks, the algorithm is available as `code/` at runtime.
Inputs are available through aliases, commonly `inputdata/stageout/file.root`.
Write retained results to `stageout/`. These are execution workspace paths;
collection puts results in **Yuki storage**, not the local task source directory.
Inline command support exists on both sides, but code-path handling differs;
consult `celebi-task-expert` before assuming `code/` means the inline task.

```bash
# From a configured task; RUNNER must match its requested runner.
celebi-cli request-runner RUNNER
celebi-cli submit --runner RUNNER --timeout 300
celebi-cli status --wait
celebi-cli log 0
celebi-cli collect plots
celebi-cli view-url
```

`submit --runner RUNNER` defaults to `local`; the runner is not a task path. Its
request timeout does not limit workflow runtime, and a timed-out request may still
have been accepted; inspect status before retrying. Submitting
from a project/directory has broader scope and checks contained tasks' runners.
Use the intended task directory for a single analysis endpoint.

New projects create `.celebi-env/default.yaml` and new tasks use the logical
environment `default` unless user configuration overrides it. Yuki registers a
native runner named `local` on first server start without replacing an existing
entry. Start its native agent and install the project environment before expecting
the default task to run; use `celebi-runner-management` for details.

## Source of truth

This revision was checked against local Celebi (HEAD `d346425`) and Yuki (HEAD
`9303c1c`), including their uncommitted changes, on 2026-10-22.
Prefer executable code and tests over older plans or CLI docstrings
when they disagree. Verify the installed version with `celebi-cli --help` and
command-specific help before using a feature absent from an older deployment.

Useful source locations in the respective repositories:

- Celebi: `CelebiChrono/celebi_cli/cli.py`, `celebi_cli/commands/` under that package,
  and `CelebiChrono/kernel/vtask*.py`.
- Yuki: `Yuki/kernel/jobs/`, `Yuki/kernel/workflows/`, `Yuki/kernel/execution/`,
  `Yuki/kernel/storage/`, and `Yuki/server/routes/`.

## Distinguish the requested operation

A task failure, an unavailable service, and data missing from a browser are different
problems. Start with the failing layer and use the routing table to load the needed
skill. Keep project UUIDs, impression UUIDs, workflow UUIDs, runner names, and runner
IDs distinct in commands and reports. For example, `stop-workflow` takes a workflow
UUID, while `view-url` takes an impression identity.

Completion means verifying the requested result: creation needs valid configuration,
execution needs terminal state and expected artifacts, a transfer needs the selected
files at the destination, and cleanup needs an account of deleted/protected items.
A generic success label or CLI exit status alone does not cover all these outcomes.
