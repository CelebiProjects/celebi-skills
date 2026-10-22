---
name: celebi-data-management
description: Register local, SSH, or XRootD data as Celebi tasks, attach existing Yuki impressions, verify data identity, and transfer stageout files between Yuki and runners.
---

# Register and move Celebi data

Use this skill for data identity and placement. For viewing plots or analyzing
collected files use `celebi-output-checker` when installed. Establish the project,
current data task/impression, source location, and intended destination first.

## Register the actual source

| Source | Operation | Effect |
|--------|-----------|--------|
| Local directory | `create-data NAME`, then `upload-data PATH` from it | Hash/register and upload data |
| Local metadata registration only | `add-source PATH` from the data task | Record the hash and impress; does not upload bytes |
| Existing Yuki impression | `attach-data IMPRESSION_UUID --path NAME` | Create/update a canonical rawdata pointer |
| Directory on an SSH runner | `register-ssh-data RUNNER REMOTE_PATH` | Hash, stage a managed runner copy, and create/fill the local pointer |
| XRootD tree visible from an SSH runner | `register-xrootd-data RUNNER ROOT_URL` | Freeze a datalist manifest without copying payloads |
| Explicit data list | `create-data-list NAME` | Datalist task to configure |
| LHCb AP query | `create-lhcb-ap-list NAME` | AP datalist task with query configuration |

Use `celebi-cli` before commands in the table. Data is represented by task flavors,
not a separate metadata type. Run registration from a project/directory to create a
pointer, or from an existing rawdata task to fill it. A normal executable task is
not a rawdata registration destination.

```bash
# From a project/directory, using real source and runner values
celebi-cli register-ssh-data farm /remote/data/run42 --descriptor run42 --timeout 120
# Then enter the resulting local rawdata task
```

After registration, inspect the resulting identity and `celebi-cli whereabouts`.
An MD5 data identifier in rawdata YAML is not interchangeable with every project,
impression, or workflow UUID. `attach-data` takes an impression UUID and verifies
that its canonical local task produces the expected identity. Investigate mismatch
messages rather than overwriting metadata by hand.

`celebi-cli verify-data --timeout 7200` recomputes the registered data hash. It can
be expensive and does not by itself repair a mismatch. Decide whether the original
content changed or the wrong location was selected before re-registering.

### XRootD manifests

Run XRootD registration from a project/directory, or from an existing XRootD-backed
datalist task. Yuki inventories files through the named SSH runner, which must have
`python3`, `xrdfs`, and the required credentials. Payload bytes are not copied.

```bash
celebi-cli register-xrootd-data farm \
  root://xrootd.example//store/sample \
  --descriptor sample --checksum auto --timeout 120
```

`--checksum auto` records server checksums when available, `required` rejects any
missing checksum, and `none` skips checksum queries. The resulting frozen manifest
uses `environment: datalist`; execution writes both `stageout/filelist.txt` and
legacy-compatible `stageout/dataList.txt`. A consumer reads the former through its
input alias. Actual XRootD authentication and reads occur in the consuming task's
runtime environment, not during ordinary manifest reuse.

## Move retained results

Use `celebi-cli whereabouts` from the intended task first. Distinguish runner
workflow stageout, managed runner cache, Yuki storage, and the client's source tree.

```bash
celebi-cli cache-results farm
celebi-cli transfer runner:farm yuki --pattern '*.root' --timeout 120
celebi-cli transfer yuki runner:farm --pattern '*.root' --timeout 120
```

Choose only the needed operation. `cache-results` copies SSH workflow outputs to
that runner's managed cache. `transfer` handles stageout files using `yuki` and
`runner:NAME` endpoints; these are service locations, not arbitrary filesystem
paths. Backend combinations have distinct implementations, including REANA paths;
check installed support and required tools rather than assuming every pair works.

Current SSH staging fences cache publication with in-progress/complete markers and
makes completed entries read-only. It can restore a missing input cache from a
same-runner producer workspace. Treat an incomplete cache as invalid; do not create
markers or copy ad hoc files into managed cache paths to make a workflow proceed.

Quote glob patterns. Existing destination files are normally skipped; `--force`
requests overwrite, not validation. Inspect transferred, skipped, and failed files,
then confirm data location and the files needed by downstream tasks. A registry
entry or zero exit code alone does not establish a complete copy. `refresh-distribution`
refreshes the registry; it is not a data transfer.

## Timeouts and partial completion

Registration/transfer default to 10 seconds per HTTP request; verification defaults
to 3600. Positive `--timeout` values apply to each request, including polls, not the
total job. Registration polling stops after 10 consecutive connection failures,
while its server job can continue. Retain returned IDs/progress information and
inspect state before retrying a mutation. Do not start duplicate uploads just
because polling ended.

For a local download, use the actual result link from `view-url`; `collect` only
brings selected results to Yuki. For archive import/export or cache deletion use
`celebi-impression-management` when available. Do not purge the only usable data
copy as a staging workaround.

Source anchors: Celebi `CelebiChrono/interface/shell_modules/object_creation.py`,
`file_operations.py`, `CelebiChrono/kernel/vtask_input.py`; Yuki
`Yuki/kernel/storage/remote.py`, `Yuki/services/result_transfer.py`, and
`Yuki/server/routes/remote_data.py`.
