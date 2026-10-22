---
name: celebi-runner-management
description: Configure and diagnose Celebi execution runners, install project-scoped Conda environments, manage compatibility mappings, and book REANA catalogs through Yuki.
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
| Project environment availability | `celebi-cli runner-environments NAME` | `celebi-cli install-environment NAME ENV` |
| Server environment mapping | `yuki env-map list` on the service host | `yuki env-map add SOURCE conda VALUE` |

Changing a client endpoint does not create a runner. Changing the global default
runner affects new tasks; it does not retarget existing tasks. `submit --runner NAME` must
match the requested runner, and an omitted argument defaults to `local`.

On first Yuki server start, the current implementation registers a native runner
named `local` if that name is absent; it preserves any existing `local` definition.
The runner entry alone is not readiness: start the host agent with
`yuki native-runner start`, then probe it. Do not re-register `local` merely because
its agent is stopped.

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
execution. REANA uses container environments. For SSH/IHEP/native, a simple logical name
that has no explicit `conda_env_map` entry resolves to
the backend's `<runner-workdir>/environments/<project_uuid>/<name>`. Each project
therefore has a separate prefix even when names match. `script`, `rawdata`, `datalist`, and
`lhcb_ap_datalist` retain their reserved behavior.

For a project-managed environment, inspect `.celebi-env/NAME.yaml`, then run:

```bash
celebi-cli install-environment RUNNER NAME
# Or select another standard Conda YAML:
celebi-cli install-environment RUNNER NAME --file PATH
celebi-cli runner-environments RUNNER
```

New Celebi projects include `.celebi-env/default.yaml` (ROOT 6.38.04, NumPy, and
Python 3.10), and new tasks default to the logical environment `default` unless
`user-config` overrides it. Install that environment per project and runner; its
presence in the source tree does not mean it is already provisioned.

The command is supported for SSH, IHEP, and native runners, creates or updates the prefix with
`conda env update --prune`, and ignores the YAML `name` field. Treat it as trusted,
mutable runner state: there is no environment-specification digest. Do not suggest it
for REANA runners. A missing project environment must be installed rather
than worked around by changing scientific configuration.

Native `<runner-workdir>` defaults to `~/.Yuki/LocalWorkflows`; its workflow state
lives below `workflows/<project_uuid>/<workflow_uuid>`. A fresh `test-runner` probe
must include native host-agent liveness, not only local tool availability.

Treat native execution and Yuki Storage as one storage domain. Successful native
stage-out is collected directly into Storage, so native has no separate managed
runner cache and is not a transfer location; use `yuki` instead. `cache-on-runner`
has no additional effect for native workflows.

On the actual Yuki host, inspect mappings with `yuki env-map list`. For a required
mapping, use `yuki env-map add SOURCE conda VALUE`, where VALUE is the runner's
usable conda environment name/path. Explicit mappings are the compatibility path and
take precedence over project prefixes; mapping a name does not install it. Although
structured entries accept a type field, the current executors consume the value as a
conda directive; do not claim that `venv` enables venv execution.

LHCb dated Conda environments are a reserved runtime form, not project-managed
prefix names: `lb-conda/PACKAGE/YYYY-MM-DD`. Yuki mounts `lhcb.cern.ch` and wraps
each user command with `lb-conda PACKAGE/DATE`; the runner/base system still needs
compatible CVMFS access. Do not combine `lb-conda/...` with a YAML `lb_run` block.

## REANA booking

Booking publishes a project file catalog to REANA and is distinct from submitting
an analysis task. For booking requests, read
[references/reana-booking.md](references/reana-booking.md).

Inspect configured runners and active use before removing a registry entry. Removal
is not cache cleanup and should not be used as a routine repair. For data placement
use `celebi-data-management`; for stale execution resources use
`celebi-impression-management` when available.

Source anchors: Celebi `CelebiChrono/celebi_cli/commands/execution_management.py`,
`task_configuration.py`, `booking.py`; Yuki
`Yuki/kernel/runners/project_environments.py`, `Yuki/kernel/workflows/ssh.py`,
`Yuki/kernel/workflows/native.py`, `Yuki/kernel/jobs/container.py`, and
`Yuki/server/routes/runner.py`.
