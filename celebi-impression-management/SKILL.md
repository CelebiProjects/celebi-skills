---
name: celebi-impression-management
description: Inspect Celebi provenance, create immutable snapshots, publish bookkeeping, migrate or archive impressions, and safely scope requested cleanup.
---

# Manage impressions and retained execution data

Use this skill for provenance, immutable snapshotting, bookkeeping publication,
snapshot storage, archive transfer, or requested cleanup. Routine execution and
file collection belong to their respective skills.
Establish the project UUID, relevant impression UUIDs, workflow UUIDs, and data
locations; these identifiers and storage layers are distinct.

## Inspect provenance

```bash
celebi-cli project-uuid
celebi-cli history
celebi-cli changes
celebi-cli search-impression PREFIX
celebi-cli trace IMPRESSION_ID
celebi-cli impression-stats
```

`trace` compares current DAG state with a stored impression. `impress` creates or
updates the current snapshot; it is not a read-only history lookup. Editing source,
parameters, or dependency state may require a new impression and affect downstream
execution. Do not manually rewrite `.celebi` refs to make old output appear current.

Local CAS snapshot objects and remote result files are different storage. A local
snapshot does not establish that runner outputs are still retained. Inspect
`whereabouts` and files for the specific impression before claiming reproducibility.

## Immutable project snapshots

Use immutable mode only for a fixed project tree whose caller prevents concurrent
edits. Run it from the project root:

```bash
celebi-cli impress --immutable
celebi-cli impress --immutable --cache-dir /persistent/cache/for-this-project
```

It evaluates the DAG once in topological order, implies offline operation, and
reuses complete CAS payloads. `--cache-dir` requires `--immutable`, belongs to one
project, and may link generated stores into the external cache. It is not a general
shared cache across unrelated projects. Do not commit those links or
`.celebi/immutable-impress*` runtime files. Conflicting non-empty local and external
stores are refused rather than merged.

`subtasks: N` is frozen inside one Task impression, not N child impressions.
Changing the repeat count changes that Task's identity; instance state and
numbered outputs belong to its execution. Do not fabricate per-index refs or
claim cross-workflow instance reuse. A generate–100 fits–summary example has
three Task impressions despite 100 fit executions.

## Publish analysis bookkeeping

Bookkeeping is an externally mutating Git-and-service workflow, not a synonym for
local `impress`. Confirm the intended repository, upstream branch, project UUID,
bookkeeping server, Yuki endpoint, commit message, and optional new tag first.

```bash
celebi-cli register-celebi-server \
  --server https://celebi.example \
  --token "$CELEBI_PROJECT_TOKEN" \
  --yuki https://yuki.example
celebi-cli bookkeep --commit "Publish results" --tag v1
celebi-cli bookkeeping-url
```

Registration verifies the analysis token and stores it in git-excluded
`.celebi/bookkeeping.local.json`. `bookkeep` authorizes before changing Git, stages
shareable changes, commits when needed, atomically pushes the upstream branch and
new tag, asks the server to impress that exact commit, then relays eligible small
collected results from Yuki with a scoped grant. Existing tags and behind/diverged
branches are rejected. Do not use it when the user only requested a local snapshot.

Missing results normally fail the publication after code has already been pushed.
Collect them and rerun `bookkeep` without `--tag` to resume the published commit.
`--allow-partial` deliberately permits missing results; it does not make them
available. Use `open-bookkeeping` only when opening a browser is requested.

## Local storage maintenance

```bash
celebi-cli migrate-impressions --dry-run --keep-legacy
celebi-cli gc-impressions --dry-run --grace-days 14
celebi-cli pack-impressions
celebi-cli tidyup
```

`migrate-impressions` defaults to execution, so explicitly request its dry run for
inspection. Review the migration and preserve legacy data until compatibility and
retention are established; use `--execute` when migration is in scope.
`gc-impressions` defaults to dry run and removes unreachable loose CAS objects past
the grace period when executed. Reachability is defined by CAS refs, not merely
which task output was most recently viewed.

The checked `pack-impressions` implementation still only evaluates thresholds and
reports stats; `packed_objects` remains zero, even with `--force`. Do not claim it
compressed data or freed disk space. `tidyup` is an interactive front end that
previews local CAS, stale workflow workspace, and SSH cache cleanup before deletion.
Keep the preview and confirmation boundaries intact. The former `homekeep` name is
obsolete.

## Export/import on the Yuki host

```bash
yuki impression-export IMPRESSION_ID --project-uuid PROJECT_UUID --output archive.tar.gz
yuki impression-import archive.tar.gz --project-uuid TARGET_PROJECT_UUID
```

Run with the correct service storage root; use `--yuki-dir` when needed. Export
includes only the named impressions present in Yuki storage; it does not fetch
missing runner data or automatically export their dependency closure. The multi-ID
export implementation can silently skip absent sources, so inspect archive members
and compare with the requested UUIDs before calling it complete.

Inspect the destination and archive contents before import, particularly when IDs
already exist; do not assume import preserves conflicting files. Verify imported
IDs and retained files afterward. An imported Yuki storage tree does not recreate
all local project objects or provision runners/environments.

## Remote cleanup when requested

`sync-live` publishes the current project live set: current task/algorithm impression
pointers plus transitive dependencies. Historical superseded impressions can be
non-live even if their history entries remain. Retention requirements may therefore
be broader than the computed live set.

```bash
celebi-cli sync-live
celebi-cli purge-stale-cache RUNNER --dry-run
celebi-cli purge-stale-workflows RUNNER --dry-run
celebi-cli stop-orphaned-workflows RUNNER --dry-run
```

Review the relevant preview, current execution, and needed copies before executing
cleanup within the user's scope. `sync-live` changes server state and is not itself
a dry run. Do not discard previews or bypass protections to reclaim space quickly.

| Operation | Scope |
|-----------|-------|
| `purge-stale-cache` | Superseded managed runner cache entries |
| `purge-stale-workflows` | Non-live, non-running runner workspaces; retains local workflow mirror |
| `stop-orphaned-workflows` | Unreferenced workflows recorded running in this project; retains workspaces |
| `stop-workflow WORKFLOW_UUID` | Explicit individual workflow cancellation, including zombie runs |
| `purge-data` | Current impression's collected Yuki files; normally refuses without a runner copy |
| `purge-ssh-runner-cache` | Broader SSH cache deletion, optionally scoped by project/impression |

`--yes` skips a CLI prompt; it does not broaden the authorized scope. `--force` on
`purge-data` can remove data that cannot be re-collected. Use broader purge commands
only when those exact data losses are intended. After execution, report removed,
skipped, protected, and failed items from the actual result.

Source anchors: Celebi `CelebiChrono/kernel/immutable_impress.py`,
`impression_gc.py`, `impression_pack.py`, `interface/shell_modules/server_booking.py`,
and `celebi_cli/commands/liveness.py`; Yuki `Yuki/kernel/storage/`,
`kernel/execution/`, `kernel/workflows/`, and `Yuki/services/celebi_server_booker.py`.
