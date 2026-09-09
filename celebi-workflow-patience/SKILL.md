---
name: celebi-workflow-patience
description: Assess slow or apparently stalled Celebi workflows using progress evidence while avoiding unnecessary resubmission, restarts, or configuration changes.
---

# Monitor a slow workflow

A running state is a reason to inspect progress, not evidence of either failure or
health. Preserve the user's configuration choices while collecting evidence.
Elapsed time alone does not justify restarting a long analysis.

```bash
celebi-cli status
celebi-cli log 0
celebi-cli engine-logs
```

These operate on the current object. Use `predecessors` for dependencies and `ls`
for effective commands. The CLI uses singular `log` and `submit`; use the installed
`--help` instead of inventing `logs`, `run`, `validate`, or `deps` commands.

## Decide from evidence

- `in movement`/`running` with advancing logs: continue monitoring at an interval
  appropriate to job duration. Avoid tight polling and keep the user informed.
- `prelude`/`waiting` or construction stages: inspect detailed status, upstream
  jobs, and runner availability. Empty user logs may be normal before execution.
- Running with no progress beyond the expected workload: read engine logs, inspect
  runner health, and compare successive observations. Read-only diagnosis is
  appropriate while running; do not blindly trust a stale status.
- `dissonance`/`failed`: diagnose construction versus execution failure before
  editing. A downstream failed job may have been skipped after upstream failure.
- `coda`/`finished`: verify retained outputs. Success does not imply the desired
  file selection is already collected into Yuki.

Native/SSH backends use per-job `.done` markers for success. Unfinished jobs can
remain unchanged while the workflow is active, then receive failure/skip details
when it terminates. SSH supervision also checks startup/exit markers and process
liveness. Do not hand-create markers or edit status files to force progress.

## Bound monitoring and interventions

`log INDEX --follow --poll-interval 5` runs until interrupted; it does not stop on
job completion. Keep follow sessions manageable, check status separately, and
report what changed or what evidence is missing. No universal two-minute waiting
period makes a restart correct.

Avoid resubmitting or changing the desired configuration merely because execution
is slow. An intentional edit creates new desired state and may leave an older
impression running; identify which run is being observed. If cancellation is within
the user's request, target the known workflow ID and confirm the resulting state
before another submission.

A timeout from registration, transfer, or submission may leave server work active.
Inspect job/impression state before retrying a mutation. Repeated connection failures
are a reason to diagnose connectivity and report uncertainty, not to launch copies.

Use `celebi-amend-expert` for a diagnosed failure and `celebi-output-checker` for
result verification when available. Source anchors: Yuki
`Yuki/kernel/native_workflow.py`, `ssh_workflow.py`, `status_constants.py`, and
`Yuki/server/routes/status.py`.

## Escalate to the relevant layer

Keep successive observations associated with the same impression/workflow. Changing
configuration while watching a task can otherwise make an older execution appear
to have disappeared. Include the last observed status and concrete log progress
when reporting a stall.

Use `celebi-runner-management` for runner probes, `yuki-service-management` for
web/worker/broker health, and `celebi-data-management` for a stalled registration or
transfer when installed. Followed logs are not an independent heartbeat, and the
current `yuki server status` implementation is not a real health probe.
