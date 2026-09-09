---
name: celebi-amend-expert
description: Diagnose and repair existing Celebi tasks and algorithms using configuration, provenance, command logs, Yuki engine status, and data-location evidence.
---

# Repair an existing Celebi workflow

Identify the task, impression, requested runner, and reported symptom. Inspect
before modifying source or configuration:

```bash
celebi-cli status
celebi-cli ls
celebi-cli predecessors
celebi-cli log 0
celebi-cli engine-logs
celebi-cli whereabouts
```

Inspect the log index corresponding to the failed user command, not only step zero.
Use `engine-logs --fetch` for a fresh backend view. `changes` and `history` help
identify edits; `trace IMPRESSION_ID` compares DAG provenance, not execution logs.
Read `.celebi/config.json` and local configuration when useful, but change managed
metadata and dependency edges through Celebi commands.

## Diagnose the layer

| Evidence | Check and repair |
|----------|------------------|
| Runner mismatch before execution | Inspect `runners` and requested runner; use `request-runner NAME` and `submit NAME` consistently |
| `dissonance` or no user logs | Inspect engine logs, workflow construction, environment mapping, input staging, and connectivity |
| `failed` with user traceback | Repair the indicated command/code/parameter issue |
| Failed downstream task with skip detail | Fix the upstream failure first; the downstream code may never have run |
| Algorithm changes appear ignored | Non-empty task commands override linked algorithm commands |
| Unsubstituted placeholder or type error | Define the value in task `parameters`; quote YAML values as strings |
| `code/code/...` or missing source | Match paths to the real source tree and runtime code link |
| Inline source missing at `code/...` | Check client versus Yuki path behavior; Yuki's code link still uses the linked algorithm |
| Duplicated input alias in path | Determine whether the command or parameter supplies the alias, and include it once |
| Completed job with absent local outputs | Check `whereabouts`, runner listings, and collection selection; outputs are remote |

## Non-obvious configuration checks

- Task commands override algorithm commands; empty/absent task commands fall back.
  Inline task environment must be non-empty. Without inline commands, a concrete
  algorithm environment must match the task; algorithm `script` permits a choice.
- Task parameter substitution does not merge algorithm defaults. Define required
  values in the task and inspect the rendered command with `ls`.
- `add-algorithm PATH` replaces the linked algorithm. For input rewiring, inspect
  the alias first and use `remove-input ALIAS` / `add-input PATH ALIAS` as needed;
  do not assume adding a duplicate alias silently replaces it.
- A source subdirectory is not intrinsically illegal; ensure command paths match.
  Avoid creating an extra `code/` wrapper to imitate the runtime mount.
- Native/SSH runners resolve environment strings through Yuki's conda mapping.
  Check `runner-envs RUNNER`, `test-runner RUNNER`, and server mappings before
  changing the scientific environment to an arbitrary image.

For backend status inconsistencies, compare detailed job status with workflow
state. Native/SSH `.done` markers indicate completion; terminal workflows classify
remaining jobs as failed or skipped. Do not manually change status files. If server
storage paths disagree, inspect the service's configured `YUKIDIR` and process
home; do not assume storage is on the client machine.

## Apply and verify

Make the smallest change supported by the evidence, preserving user configuration
and unrelated work. Edit code/YAML in the owning object, use CLI operations for
links and settings, and inspect `changes`, `ls`, and `status` afterward. Run focused
syntax or logic checks where useful. If rerunning is within scope, submit from the
affected task with its requested runner, inspect the relevant logs, and verify the
expected stageout artifacts and scientific result.

A stale listing can be refreshed with `refresh-filelists`; selected files can be
collected with `collect '*.root'`. Neither operation requires changing the algorithm.
Do not delete caches, purge impressions, or restart active workflows as a generic
repair. A client timeout does not establish that the server job stopped.

Use `celebi-algorithm-expert`, `celebi-task-expert`, or `celebi-output-checker` for
the relevant work when installed. Source anchors: Celebi
`CelebiChrono/kernel/vtask_setting.py`, `vtask_input.py`, `vobj_execution.py`, and
Yuki `Yuki/kernel/container_job.py`, `native_workflow.py`, `ssh_workflow.py`.

## Repair the owning layer

Route unavailable runners/environments to `celebi-runner-management`, registration
or staging failures to `celebi-data-management`, and web/worker/storage-root failures
to `yuki-service-management` when installed. In particular, do not trust the current
placeholder `yuki server status` as evidence that both Flask and Celery work.

For missing historical results or requested cleanup, use
`celebi-impression-management`. Its liveness and retention rules are separate from
scientific validity. Preserve a useful failing impression and diagnostic evidence
before a repair changes the current desired state.
