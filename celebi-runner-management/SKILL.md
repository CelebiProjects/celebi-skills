---
name: celebi-runner-management
description: Configure and diagnose Celebi execution runners, requested-runner settings, conda environment mappings, and REANA booking through Yuki.
---

# Manage execution runners

Use this skill when registering or changing a runner, diagnosing backend capability,
or setting up REANA booking. Ordinary task submission belongs to
`celebi-workflow-execution`; starting the service belongs to `yuki-service-management`
when those skills are installed.

## Identify the configuration layer

| Layer | Inspection | Change |
|-------|------------|--------|
| Client's Yuki endpoint | `celebi-cli dite` | `celebi-cli set-dite URL` |
| Yuki runner registry | `celebi-cli runners` | `register-runner` / `update-runner` |
| Current object's requested runner | `celebi-cli status` | `celebi-cli request-runner NAME` |
| Creation defaults | `celebi-cli user-config --list` | `celebi-cli user-config` |
| Runner environment availability | `celebi-cli runner-environments NAME` | Provision in the intended runner environment |
| Server environment mapping | `yuki env-map list` on the service host | `yuki env-map add SOURCE conda VALUE` |

Changing a client endpoint does not create a runner. Changing the global default
runner affects new tasks; it does not retarget existing tasks. `submit --runner NAME` must
match the requested runner, and an omitted argument defaults to `local`.

## Configure the backend actually in use

Inspect the existing name, backend, URL, settings, and health before updating it.
Supported workflow backends include `reana`, `native`, and `ssh`; old help examples
such as `docker` or `slurm` are not evidence of a supported backend selector.

`celebi-cli register-runner NAME URL SECRET BACKEND_TYPE [OPTIONS]` creates a registry
entry. Use `update-runner NAME` to change only necessary settings of an existing one.
Read command-specific help for available options. Keep credentials out of reports
and use the existing credential mechanism rather than inventing example secrets.

For an existing SSH runner, a settings update might be:

```bash
celebi-cli update-runner farm --ssh-host compute.example.org --ssh-user analyst --remote-workdir /scratch/analyst/yuki --cores 8
celebi-cli test-runner farm --timeout 30
celebi-cli runner-environments farm
celebi-cli runners
```

Substitute actual authorized values. SSH key paths must be usable by the **Yuki
process**, not just the client shell. `--remote-workdir`, `--conda-path`, and
`--snakemake-path` refer to the runner host for SSH execution. Native `--workdir`
refers to the service-side execution host. REANA needs its endpoint and access
credentials; Kerberos/EOS settings apply only when required by that deployment.

`test-runner` probes capabilities and persists health information. It is not a
scientific workflow test. A saved healthy status may be stale; use a fresh probe
when diagnosing a new connection or environment failure.

## Resolve environments

The algorithm/task YAML environment and backend implementation jointly determine
execution. REANA uses container environments. Native/SSH map environment identifiers
to conda directives using Yuki's `conda_env_map`: exact key, then Docker-prefix
stripping and retry, then a sanitized fallback name. `script` skips conda activation.

On the actual Yuki host, inspect mappings with `yuki env-map list`. For a required
mapping, use `yuki env-map add SOURCE conda VALUE`, where VALUE is the runner's
usable conda environment name/path. Verify with `runner-environments` and a task when
execution is authorized. Mapping a name does not install the environment. Although
structured entries accept a type field, the current executors consume the value
as a conda directive; do not claim that `venv` enables venv execution.

## REANA booking

Booking publishes a project file catalog to REANA and is distinct from submitting
an analysis task. For booking requests, read
[references/reana-booking.md](references/reana-booking.md).

Inspect configured runners and active use before removing a registry entry. Removal
is not cache cleanup and should not be used as a routine repair. For data placement
use `celebi-data-management`; for stale execution resources use
`celebi-impression-management` when available.

Source anchors: Celebi `CelebiChrono/celebi_cli/commands/execution_management.py`,
`task_configuration.py`, `booking.py`; Yuki `Yuki/kernel/runner_config.py`,
`native_workflow.py`, `ssh_workflow.py`, and `Yuki/server/runner_probe.py`.
