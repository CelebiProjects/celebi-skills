---
name: celebi-task-expert
description: Configure Celebi tasks with linked algorithms or inline commands, repeated execution with subtasks, indexed inputs and outputs, parameters, environments, and runners.
---

# Configure a Celebi task

Create with `celebi-cli create-task NAME` under a project or directory, then work
from the task directory. Inspect existing `celebi.yaml`, `ls`, and `status` before
editing. Use CLI operations for dependency/metadata changes; source and task YAML
are editable. Keep a short README describing inputs and retained outputs.

## Select the command source

Effective commands resolve in this order on both Celebi and Yuki:

1. Non-empty task `commands`.
2. Linked algorithm `commands`.
3. No commands.

For a reusable algorithm, run `celebi-cli add-algorithm PATH`. It replaces an
existing algorithm link, but does not remove inline commands. To switch back to
algorithm commands, explicitly remove or empty the task's command list.

Inline tasks may have their own source and commands without an algorithm. A
self-contained example that avoids backend code-path differences is:

```yaml
environment: script
memory_limit: 256Mi
parameters:
  message: "hello"
commands:
  - mkdir -p stageout && printf '%s\n' "${message}" > stageout/message.txt
```

This requires a native/SSH runner with the necessary shell tools; choose a real
container environment for REANA. Non-empty inline commands require a non-empty
task environment. `rawdata`, `datalist`, and `lhcb_ap_datalist` are special data
flavors and do not require a linked algorithm.

### Inline source path limitation

The client chooses the task directory as `code_path()` for inline commands and
mounts it as `code/` in Docker tests/workarounds. In the checked Yuki implementation,
`ContainerJob._create_symlink_commands()` creates `code` only for a linked algorithm,
and `_substitute_paths()` resolves `${code}` only to that algorithm. Task commands
win, but these ordinary-task runtime paths do not switch to the task. Do not promise that
`python code/script.py` accesses inline task code on every backend. Inspect the
generated workflow/staging for the chosen backend; use a linked algorithm for
portable source-file execution if that meets the user's task design.

Repeated inline tasks with `subtasks: N` do expose task source as `code/` in Yuki;
their workspaces and path substitution differ from ordinary inline execution.

## Repeat one Task with subtasks

Set `subtasks: N` in task YAML for simple repetition; keep `object_type: task`.
N is a positive integer, and `${index}` supplies integers 0 through N - 1.
Do not add a user `parameters.index`, create N Task directories, or generate N
impressions for this model. `create-multi-tasks` remains a different operation.

```yaml
environment: python3.12
subtasks: 100
commands:
  - python code/fit.py --index ${index} --input toys/stageout/${index}/*.json --output stageout/index
```

Use `${index}` for indexed inputs and literal `stageout/index/` for outputs.
The runner maps the latter directly to `stageout/<number>/`; cache, collection
and downstream aliases retain the numeric layout. An alias references the whole
upstream Task, and downstream Tasks wait for all instances. `subtasks: 1` still
uses `/0/`; absent `subtasks` preserves ordinary behavior.

Read [references/subtasks.md](references/subtasks.md) when configuring repeats,
adapting an algorithm's output paths, or building the 100-toy example.

## Configure the execution context

```bash
celebi-cli set-parameter events 10000
celebi-cli add-input @/tasks/source inputdata
celebi-cli set-environment ENVIRONMENT
celebi-cli set-memory-limit 4Gi
celebi-cli request-runner RUNNER
celebi-cli ls
celebi-cli status
```

`add-input PATH ALIAS` declares a dependency; `remove-input ALIAS` removes it.
Do not hand-write `dependencies` or `alias` YAML as a replacement for graph edges.
Use `predecessors` and `successors` to inspect links.

For numbered dependencies, `add-multi-inputs PATH_BASE ALIAS_BASE STOP
--start START` adds `PATH_BASE_N` as `ALIAS_BASE_N` for the half-open range
`START <= N < STOP`. `remove-multi-inputs ALIAS_BASE STOP --start START` removes
the corresponding aliases. Review the range first; each command permits at most
10000 items and may leave earlier changes in place if a later item fails.

Task YAML may contain `descriptor`, `environment`, `memory_limit`, `parameters`,
inline `commands`, optional `subtasks`, and an optional `lb_run` mapping. These are
not an exhaustive schema; preserve supported backend/data fields already in use.
Current Yuki converts numeric values to text before substitution; quote values
according to their intended shell usage. There is no automatic
merge of algorithm parameter defaults into the task.

Algorithm-backed tasks must match a concrete algorithm environment; an algorithm
with `environment: script` permits the task's chosen environment. New-task defaults
come from `user-config` at creation time. Changing global defaults does not rewrite
existing task YAML. Use `celebi-cli user-config --list` to inspect effective
creation defaults; `user-config` edits the user's settings, while `config` edits the
current object's YAML. Do not substitute global settings changes for configuring an
existing task.

New projects currently create `.celebi-env/default.yaml`, and new tasks default to
the logical environment `default`. Provision it on the intended native/SSH/IHEP
runner with `install-environment`; changing the task to an unrelated image is not a
substitute. `lb-conda/PACKAGE/YYYY-MM-DD` is a separate reserved environment form.
Do not combine it with `lb_run`. If task commands are non-empty, the task-level
`lb_run` contract replaces the linked algorithm's along with its commands.

## Paths, submission, and results

A producing task's retained files are commonly at `ALIAS/stageout/FILE` in the
consumer workspace. A parameter is relative to the alias only if the command
prepends that alias. Match the actual command to avoid duplicate path prefixes.

Parameter, code, or dependency edits can change impressions and downstream state;
inspect `changes`/`status` and resubmit when needed. `celebi-cli submit --runner RUNNER` uses
the current object and checks its requested runner; it does not take a task path.
Write results to runtime `stageout/`. `collect` copies results to Yuki, not to this
source directory.

For runner setup and monitoring use `celebi-workflow-execution`; for results use
`celebi-output-checker` when available. Implementation references: Celebi
`CelebiChrono/kernel/vtask.py`, `vtask_setting.py`, `vtask_input.py`, and Yuki
`Yuki/kernel/jobs/container.py`.

## Verify input readiness without changing identity

Before submission, inspect the producer's expected impression, file selection, and
`whereabouts`. A valid alias edge does not guarantee the selected runner can access
its data. Use `celebi-data-management` for registration/transfer and
`celebi-runner-management` for runner capabilities when installed. An inaccessible
cached input should be diagnosed at its data location before replacing the dependency
or rerunning an expensive producer.
