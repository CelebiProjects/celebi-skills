---
name: celebi-workflow-execution
description: Submit and monitor configured Celebi workflows through Yuki, verify execution scope and backend states, and recover from submission uncertainty.
---

# Execute a Celebi workflow

Work from the intended task or container object. Inspect `celebi-cli ls`, `status`,
and `predecessors` for commands, inputs, and requested runner before submission.
Algorithms are not submitted directly. Project/directory submission includes
contained tasks and checks that their requested runners agree.

```bash
celebi-cli dite
celebi-cli runners
celebi-cli request-runner RUNNER
celebi-cli submit --runner RUNNER --timeout 300
celebi-cli status --wait
celebi-cli log 0
```

`submit --runner RUNNER` defaults to `local` when the option is omitted, even when
a different runner is configured. `request-runner` sets the target;
submission rejects a mismatch. Dependencies are resolved as part of execution.
Verify the success message and subsequent status; a command returning without a
shell error is not alone proof that submission succeeded.

`--timeout` is the submission HTTP timeout, not a workflow-runtime limit. Yuki also
uses an explicit longer value to extend SSH connection/command/start-confirmation
limits for that workflow. A timed-out request may still be accepted. Current Yuki
uses execution leases and returns an existing active/completed workflow for the
same impressions where appropriate, but the client must still inspect status before
retrying rather than rely on deduplication as a general retry mechanism.

## Confirm runner readiness

Inspect `celebi-cli runners` and the task's environment. A runner name such as
`local` does not identify the backend; native/SSH may map image identifiers to conda
environments while REANA uses containers. For registration, capabilities, or mapping
changes, use `celebi-runner-management` if available. Use `yuki-service-management`
for service/worker faults rather than restarting it as part of ordinary submission.

## Monitoring and status

For `subtasks: N`, Yuki expands one Task impression into N indexed Snakemake
executions and an aggregate completion rule. Downstream Tasks and full cache
publication wait for all instances. Use `log 0 --subtask 7` for command 0 of
instance 7; ordinary `log` index still means command number. Successful instances
can be preserved by Snakemake in the same workspace, but a new Celebi workflow
may run all N again. There is no cross-workflow per-index reuse or index retry
CLI. Do not remove failed-workflow cache fences to force a retry.

Use `status` and `log INDEX` for task progress, `engine-logs` for orchestration
failures, and `engine-logs --fetch` when a fresh backend fetch is needed.
`status --wait` refreshes runner state until terminal completion; use
`--poll-interval SECONDS` (or `-i`) to change its two-second default. It is a
blocking monitor, so bound it according to the user's request.
`log 0 --follow --poll-interval 5` follows the first user command's log; it runs
until interrupted, so keep the session bounded and check status separately.

| Status names that may appear | Meaning |
|--------------------------------|---------|
| `new` | Local object/impression state changed or needs execution inspection |
| `silence` / `raw` | Initial execution state |
| `prelude` / `waiting` | Waiting/preparation |
| `composing` / `submitted`, `orchestrating` / `built`, `tuning` / `ready` | Construction/setup stages |
| `in movement` / `running` | Active execution |
| `coda` / `finished` / `success` | Successful completion |
| `dissonance` | Workflow construction failed |
| `failed` | Backend execution failed; may include a skipped downstream job |
| `stopped`, `deleted`, `final note` / `archived` | Stopped, removed, or archived state; not fresh execution success |

Read the detailed status, not just one label. Native/SSH backends mark jobs complete
from `.done` markers. When a workflow ends, unfinished jobs may become failed with
a log tail or an upstream-dependency skip message. Missing markers during active
execution alone do not establish failure. Use `celebi-workflow-patience` or
`celebi-amend-expert` when available.

## Data placement

Outputs live in runner workspaces/caches and/or Yuki storage. `collect` brings
selected outputs into Yuki; it does not download to the local task source folder.
Use `whereabouts` before concluding data is missing.

```bash
celebi-cli whereabouts
celebi-cli collect plots
celebi-cli collect '*.root'
celebi-cli cache-results RUNNER
celebi-cli transfer runner:RUNNER yuki --pattern '*.root' --timeout 120
```

Choose only the operations needed: `cache-results` preserves SSH workflow results
in that runner's managed cache; `transfer` moves stageout data between `yuki` and
`runner:NAME` locations. Inspect command output for partial failures. Quote glob
patterns. Use `--force` only when overwriting destination files is intended.

Registration and transfer HTTP timeouts default to 10 seconds; verification defaults
to 3600. A positive `--timeout` changes each request's timeout, not total job duration.
A client timeout does not prove the server stopped. Inspect state before resubmitting
or starting duplicate transfers.

## Cancellation and recovery

For requested cancellation, `stop-workflow WORKFLOW_UUID` targets an execution,
not an impression. Inspect the resulting state before another submission. Bulk
cancellation and stale-resource cleanup have different liveness scopes; use
`celebi-impression-management` when available instead of treating them as retries.

After a disconnected submission, retain the task/impression, runner, and any workflow
ID already returned. Check status and engine logs to distinguish a rejected request
from work accepted by Yuki. Report uncertainty if connectivity prevents that check;
do not launch duplicate jobs to obtain a clearer success message.

If the recorded workflow state is specifically unknown, `submit --runner RUNNER
--with-unknown` asks Celebi to refresh that workflow before deciding whether a new
submission is needed. Do not use it as a generic force-resubmit flag.

`celebi-cli cancel` cancels running impressions in the current task or container
scope without deleting finished results or configuration. Use
`stop-workflow WORKFLOW_UUID` only when targeting a known individual execution.
The older `kill` and `kill-workflow` names are compatibility aliases, not preferred
commands.

After terminal success, verify the artifacts needed by the next step with
`celebi-output-checker`. Use `celebi-data-management` for complete data registration
or transfer procedures. Report whether execution, collection, and result validation
have each completed; one does not imply the others.

For a failed workflow, `collect --allow-failed SELECTOR` may retrieve diagnostic
stageout that already exists; it does not convert the execution to success.

Implementation anchors: Celebi `CelebiChrono/kernel/vobj_execution.py` and
`celebi_cli/commands/` under that package; Yuki `Yuki/kernel/execution/`,
`kernel/workflows/`, and `kernel/storage/`.
