---
name: celebi-algorithm-expert
description: Create or edit reusable Celebi algorithm code and command templates, including parameter substitution, runtime paths, and retained outputs.
---

# Implement a Celebi algorithm

Inspect the existing algorithm and its consuming tasks before changing their
contract. Create an algorithm with `celebi-cli create-algorithm NAME` from a project
or directory object. Edit its source and `celebi.yaml`; keep its README concise.
Use Celebi commands for `.celebi` metadata changes.

## Commands and parameters

A minimal reusable template is:

```yaml
environment: script
commands:
  - python code/analyze.py --events "${events}" --output stageout/results.root
```

`script` on an algorithm allows the consuming task to choose its environment.
For an algorithm declaring a concrete environment, the client's validation requires
the task's environment to match it. Set the actual execution environment on the
task; linking an algorithm does not automatically copy it.

The executor substitutes values from the **task's** `parameters` mapping. Set every
required value there, normally with `celebi-cli set-parameter events 10000`. When
editing YAML directly, current Yuki converts numeric values to text before
substitution; quote values for their intended shell usage. Algorithm `parameters` are not
implicitly merged into the task.

Commands are shell strings. Quote values according to their intended shell usage;
substitution is textual, not argument escaping. Do not reuse parameter names for
reserved path placeholders (`code`, `workspace`, `output`) or input aliases.

For LHCb releases, an algorithm may add an `lb_run` mapping with required string
fields `project` and `version`, plus optional `platform`; these fields support task
parameter substitution. Yuki then wraps only the user commands with `lb-run` and
adds the `lhcb.cern.ch` CVMFS requirement. The normal `environment` remains the
base container/Conda environment and must be ABI-compatible. Alternatively,
`environment: lb-conda/PACKAGE/YYYY-MM-DD` selects a dated LbConda environment.
The two mechanisms cannot be combined.

## Runtime filesystem

For an algorithm-backed task, the algorithm directory is linked as `code/`.
`analyze.py` in the algorithm root is therefore `code/analyze.py`. A package or
source subdirectory is valid if commands use its real relative path. Do not add a
wrapper `code/` directory just to match the runtime link.

Retained results belong in `stageout/`; temporary intermediates may live elsewhere
in the workspace. Keep input data read-only and access it through declared aliases.
For example:

```yaml
environment: script
commands:
  - python code/analyze.py --input "inputdata/${input_file}" --output stageout/results.root
```

Here the task supplies `input_file: "stageout/data.root"`. If the command instead
uses `${input_file}` directly, the parameter must contain the full runtime path.
The path contract depends on the command; avoid prepending the alias twice.

For a consuming Task with `subtasks: N`, accept the injected `${index}` for
indexed inputs, e.g. `toys/stageout/${index}/*.json`, and write through literal
`stageout/index/`. The runner maps this to the Task's `stageout/<number>/`.
Keep repetition on the Task, not the Algorithm; do not put `subtasks` or a
user `index` default on the Algorithm to control the workflow. An algorithm
used by ordinary and repeated tasks may accept an output-directory parameter
whose value is `stageout/` or `stageout/index/` respectively.

## Integration and validation

From a consuming task, link with `celebi-cli add-algorithm @/algorithms/analyze`,
configure parameters/environment, then inspect `celebi-cli ls` and `status`.
A task's non-empty `commands` override the algorithm's commands; check for those
before diagnosing why algorithm edits have no effect.

Use syntax checks and focused tests where they meaningfully validate the change.
Celebi does not automatically test scientific logic or install arbitrary package
dependencies. Validate execution in the intended runner/environment when execution
is in scope, and inspect outputs before claiming the analysis succeeded.

For inline task code, use `celebi-task-expert` if available: client test/workaround
mounts and the current Yuki runtime do not have identical inline code-path behavior.
Source anchors: Celebi `kernel/vtask_setting.py`, `vtask.py` under
`CelebiChrono/`, and Yuki `Yuki/kernel/jobs/container.py`.

## Preserve reproducibility across consumers

Inspect `celebi-cli successors` before changing a shared algorithm's interface.
Keep existing parameter/input conventions compatible, or update the affected tasks
within the requested scope and report that change. Source edits may create new
impressions for multiple consumers; do not submit all of them merely to test one.
Validate with a representative intended task and its actual environment when a run
is requested. Use `celebi-runner-management` for unavailable environments and
`celebi-impression-management` for snapshot/provenance questions if installed.
