# Repeated Task execution

Use this reference for `subtasks: N`, indexed input/output paths, instance logs
or the three-Task Gaussian toy example. The contract requires updated Celebi
and Yuki; verify the deployed version before submitting work.

## Definition and identity

`subtasks` is a positive YAML integer. Explicit null, booleans, non-integers,
zero and negative values are invalid. `${index}` is substituted as the decimal
representation of an integer in `[0, N)`, suitable for `argparse type=int`.
The task's own parameters must not define `index` when `subtasks` is present;
ordinary tasks may still use an index parameter.

The object remains a single `Task`/`VTask` with one frozen impression, shared
code, environment, parameters and input aliases. There is no `VBulkTask`,
parameter matrix, per-instance override mapping or per-index impression.
Changing N changes the Task impression. `create-multi-tasks` instead creates
independent filesystem Task objects and sets their ordinary index parameters.

## Paths

| Location | Path |
|---|---|
| Indexed input selected in a command | `toys/stageout/${index}/*.json` |
| Output written by one instance | literal `stageout/index/fit.json` |
| Output in online cache or collected storage | `stageout/<number>/fit.json` |
| The same output through downstream alias `fits` | `fits/stageout/<number>/fit.json` |

Do not change the output path to `stageout/${index}/...`: the fixed directory
name `index` is the supported instance output interface. A link maps it directly
to the current numeric partition; no output copy/rename is required. Even
`subtasks: 1` uses numeric partition 0. Without `subtasks`, preserve normal
`stageout/...` output paths.

Each instance has a separate working directory. `code/` points to the algorithm,
or to the repeated inline Task's shared source when no algorithm is linked.
For repeated commands `${workspace}`/`${output}` resolve to the instance workspace
and `${code}` to `code`. Ordinary inline Task path limitations still apply outside
this execution path. Treat code and input aliases as read-only; working-directory
separation is not a host filesystem security sandbox.

An alias exposes the entire upstream Task. Use one normal `add-input` edge,
without per-index aliases or automatic index remapping. Downstream Tasks wait
for the upstream aggregate completion marker. A normal producer may explicitly
write numbered partitions without declaring subtasks. An unquoted glob in a shell
command expands before the script runs; agree whether a partition contains one
file or a script accepts several.

## Three-Task Gaussian example

The local example is `Demo/demo-subtasks-01-gaussian-toys`:

1. GenerateTask runs once, generates 100 independent toys with 1,000 observations
   each, and writes `stageout/<number>/toy.json`. Truth is mean 5 and sigma 1.5;
   seed is 20261025 + index.
2. FitTask declares `subtasks: 100`, reads `toys/stageout/${index}/*.json` and
   writes literal `stageout/index/fit.json`. Fits use the exact unbinned Gaussian
   maximum-likelihood solution and asymptotic curvature errors, using Python's
   standard library.
3. SummaryTask requires indices 0 through 99 under `fits/stageout/` and produces
   `fits.csv`, `summary.json`, `summary.svg` and `report.md`. Missing, failed or
   inconsistent results must fail the summary rather than silently shrink it.

Connect the Tasks with `add-input ../GenerateTask toys` from FitTask and
`add-input ../FitTask fits` from SummaryTask. Use `environment: python3.12` and
a project Conda specification containing Python 3.12. Install that environment
on the chosen runner, set all requested runners consistently, and submit the
SummaryTask to resolve the chain. Update toy count, subtasks and expected summary
count together when changing ensemble size.

The Demo's `run_local.py --cores 4` materializes three real impressions and runs
Yuki's production Snakefile using the current environment, without a live
service. Its `.local-run/validation.json` and results are evidence of local
validation only. Do not report a remote deployment or online run from that alone.

## Monitoring, cache and recovery

`log 0 --subtask 7 [--follow]` selects user command 0 from execution 7. Command
index and subtask index are different. Logs and status files are under
`logs/<number>/`. Status files record running/failed/finished; missing means
not started, and an abrupt termination may leave running stale.

The Task completion marker and full cache are published after every instance
succeeds. Snakemake `--rerun-incomplete` can preserve successful instances in
the same workspace. A new Celebi workflow may run all instances again; do not
promise cross-workflow per-index reuse or invent a single-index retry CLI.
Preserve existing execution leases and failed-workflow cache fencing.

Data-only environments and `htcondorcern` reject subtasks. Client docker-test,
ssh-test and workaround paths do not expand repeats and explicitly reject them;
use workflow submission or the Demo's independent validator.

Source anchors: Celebi `utils/subtasks.py`, `kernel/vtask_setting.py`,
`kernel/vimpression.py`; Yuki `kernel/jobs/container.py`,
`kernel/workflows/base.py` and `UnitTest/test_subtasks.py`.
