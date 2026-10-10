---
name: celebi-impression-management
description: Inspect Celebi impression provenance, migrate or assess local snapshot storage, export/import Yuki impressions, and safely scope requested cache or workflow cleanup.
---

# Manage impressions and retained execution data

Use this skill for provenance, snapshot storage, archive transfer, or requested
cleanup. Routine execution and file collection belong to their respective skills.
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

## Local storage maintenance

```bash
celebi-cli migrate-impressions --dry-run --keep-legacy
celebi-cli gc-impressions --dry-run --grace-days 14
celebi-cli pack-impressions
```

`migrate-impressions` defaults to execution, so explicitly request its dry run for
inspection. Review the migration and preserve legacy data until compatibility and
retention are established; use `--execute` when migration is in scope.
`gc-impressions` defaults to dry run and removes unreachable loose CAS objects past
the grace period when executed. Reachability is defined by CAS refs, not merely
which task output was most recently viewed.

The checked `pack-impressions` implementation only evaluates thresholds/stats;
`packed_objects` remains zero. Do not claim it compressed data or freed disk space,
even with `--force`.

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

Source anchors: Celebi `CelebiChrono/kernel/impression_gc.py`, `impression_pack.py`,
`liveness.py`, `celebi_cli/commands/liveness.py` under that package; Yuki
`Yuki/kernel/impression_transfer.py`, `workflow_purge.py`, and `workflow_kill.py`.
