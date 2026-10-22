---
name: celebi-navigation
description: Explore Celebi analysis objects, project paths, containment, and dependency graphs before configuring or debugging workflows.
---

# Navigate a Celebi project

Start in the actual project or object directory. For standalone CLI invocations,
use shell `cd` or the tool's working-directory option. Use `project-path` to print
the current project path and `use-project NAME` to select a registered project for
later Celebi sessions. The compatibility names `navigate`, `cdproject`, and
standalone `celebi-cli cd` cannot move their parent shell and should not be used in
new instructions. Interactive Celebi shell navigation maintains its own context.

```bash
celebi-cli tree
celebi-cli ls
celebi-cli short-ls
celebi-cli status
celebi-cli project-uuid
```

Use `celebi-cli ls [path]` for object-aware listings. Read source/configuration with
normal filesystem tools when useful; an ordinary file listing does not identify
Celebi object types.

## Project registration and identity

`list-projects` shows the per-user registry, `use-project NAME` selects an entry,
and `register-project PATH` adds an existing project without moving its files.
Registering an ordinary checkout preserves its project UUID. For a copied project
that is intentionally a distinct analysis, use `register-project PATH --new-project`
(or interactive `celebi use PATH --new-project`) to write a fresh UUID. This is an
identity change, not a harmless navigation flag; do not use it merely to open a
second checkout of the same project.

When initializing for an analysis already created by the bookkeeping website, use
`celebi-cli init --uuid ISSUED_UUID`. Otherwise `init` generates a UUID. Confirm the
intended identity before registration or publication.

## Containment and dependency graphs

Projects and directories contain objects; directories may contain directories.
Algorithms contain reusable code. Tasks either link an algorithm or carry their
own commands. Data objects are task flavors. Read `object_type` in
`.celebi/config.json` when identity is unclear; leave metadata mutations to the CLI.

`tree` is the containment tree, not a substitute for inspecting dependency edges:

```bash
celebi-cli predecessors
celebi-cli successors
celebi-cli draw-dag analysis.svg --exclude-algorithms
```

`draw-dag` needs Graphviz and saves under the `dag_output_dir` user setting
(default `~/Downloads/`). Inspect `celebi-cli user-config --list` for the effective path. Include algorithms when
investigating code dependencies. Task `ls` shows configuration and effective
commands; read `celebi.yaml` to distinguish inline commands from algorithm fallback.

## Paths

- In `add-input` and `add-algorithm`, `@/` anchors to the project root. Ordinary
  relative paths are relative to the current object context.
- Shell `cd` requires a real filesystem path, not `@/`.
- Runtime `code/` and input aliases are workspace paths, not local directory names
  to create for navigation.
- `celebi-cli edit code:script.py` resolves through the task's effective code root
  (the task for inline commands, otherwise its algorithm).

To inspect provenance, use `history`, `changes`, and `trace IMPRESSION_ID`.
`trace` compares dependency DAGs with a stored impression; it is not a log viewer.
`search-impression PREFIX` helps resolve an impression identity.

Use `celebi-object-creation` for new objects, `celebi-task-expert` for links and
parameters, and `celebi-amend-expert` for diagnosed failures when those skills are
available. The authoritative command definitions are in Celebi's
`CelebiChrono/celebi_cli/commands/`; path behavior is covered by
`UnitTest/test_task_configuration_paths.py`.

## Follow the right identity

Use `project-uuid` for the project namespace and `search-impression`/`history` for
snapshots. Workflow IDs identify executions on a backend; runner names select
registry entries. Do not treat an impression prefix as a workflow cancellation ID.
For deeper provenance, archive, or storage questions load
`celebi-impression-management` if available. For the location of actual result bytes,
use `whereabouts` and `celebi-data-management`; a source-tree path does not locate
remote data.
