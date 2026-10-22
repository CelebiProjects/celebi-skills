---
name: celebi-object-creation
description: Create and link Celebi project objects, including nested directories, algorithms, executable tasks, and local or remote data tasks.
---

# Create Celebi objects

Inspect the intended parent with `celebi-cli ls` before creation. Projects and
directories can contain tasks, algorithms, and nested directories. Use Celebi
creation commands so metadata is initialized correctly.

| Command | Result |
|---------|--------|
| `celebi-cli mkdir NAME` | Directory object; nesting is supported |
| `celebi-cli create-algorithm NAME` | Reusable code object |
| `celebi-cli create-task NAME` | Task with creation-time user defaults |
| `celebi-cli create-data NAME` | Rawdata task |
| `celebi-cli create-data-list NAME` | Datalist task |
| `celebi-cli create-lhcb-ap-list NAME` | LHCb AP datalist task; configure its query |

Data flavors have metadata `object_type: task`. A normal task can execute a linked
algorithm or its own non-empty YAML `commands`. Do not require every task to have
an algorithm. Use ordinary filesystem operations for source files, and CLI `mv` or
`cp` for object moves/copies so graph relationships are maintained.

## Algorithm-backed task

From a project or directory:

```bash
celebi-cli create-algorithm analyze
celebi-cli create-task analysis
cd analysis
celebi-cli add-algorithm ../analyze
celebi-cli set-parameter events 10000
```

Implement the algorithm commands/source and set the task environment before
submission. Linking does not copy parameter defaults or environment. Use
`celebi-algorithm-expert` and `celebi-task-expert` if available.

For a consumer of an existing producer:

```bash
# From the consumer task
celebi-cli add-input @/tasks/producer inputdata
celebi-cli predecessors
celebi-cli ls
```

The second argument is an alias. Do not substitute manual dependency JSON for this
operation. `@/` paths are project-root paths supported by these linking commands.

For numbered objects or dependencies, use the bounded half-open range commands:

```bash
celebi-cli create-multi-tasks simulation 20 --start 10
celebi-cli add-multi-inputs @/tasks/source input 20 --start 10
```

The first creates `simulation_10` through `simulation_19` and sets each task's
`index` parameter. The second links `source_10` through `source_19` under matching
`input_N` aliases. Use `remove-multi-inputs ALIAS_BASE STOP --start START` to remove
the same alias range. A bulk invocation is limited to 10000 items.

## Data sources

Choose the operation matching the actual location of the bytes:

- Local data: create a data task, enter it, and use `celebi-cli upload-data PATH`.
- Local directory collection: `celebi-cli create-multi-data PATH` creates and
  uploads one data object per immediate child directory.
- Existing Yuki impression: `celebi-cli attach-data IMPRESSION_UUID --path NAME`
  creates a local rawdata pointer.
- Data on an SSH runner: `celebi-cli register-ssh-data RUNNER /remote/data
  --descriptor NAME --timeout 120` computes metadata, stages data in the runner's
  managed impression area, and creates or fills a local pointer task.
- XRootD tree visible from an SSH runner: `celebi-cli register-xrootd-data RUNNER
  ROOT_URL --descriptor NAME --checksum auto --timeout 120` freezes a datalist
  manifest without copying payloads. It is valid from a project/directory or an
  existing XRootD-backed datalist, not an unrelated task.

Registration copies/stages remote data and may be lengthy; it is not just a local
metadata edit. Prefer it over generating a dummy computation to represent existing
external data. Check `whereabouts` from the data task afterward. Use `verify-data`
when content verification is needed; it recomputes the data hash.

`--timeout` for registration and verification is per HTTP request, not a total job
limit. Registration polling retries transient connection failures and stops after
10 consecutive failures; the server job can continue. Inspect state before retrying
a registration whose polling ended without a result.

After creation, check object type, effective commands, inputs, environment, and
README. Do not claim creation implies execution readiness or successful execution.
Source anchors: Celebi `CelebiChrono/kernel/vdirectory.py`, `vtask.py`,
`celebi_cli/commands/object_creation.py` under the same package, and Yuki
`Yuki/kernel/storage/remote.py`.

## Choose the next operation

Object creation does not provision the service, install environments, or make all
input bytes available. Use `celebi-data-management` for the full registration and
verification workflow, `celebi-runner-management` for execution capability, and
`yuki-service-management` for an unavailable DITE service when installed.

For organization changes, inspect existing object names and graph edges before
`mv`/`cp`; ordinary directory copies can leave incorrect object identity and arcs.
Document the new object's expected input/output contract rather than copying
historical runtime outputs into its source tree.
