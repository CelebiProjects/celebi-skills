---
name: celebi-output-checker
description: Inspect Celebi task logs and scientific results, collect selected runner outputs into Yuki, and retrieve files for local viewing or analysis.
---

# Inspect Celebi results

Set the working directory to the intended task and establish the impression and
execution state before interpreting results:

```bash
celebi-cli status
celebi-cli log 0
celebi-cli whereabouts
celebi-cli view-url
```

`view-url [IMPRESSION]` and `view [IMPRESSION]` accept an impression ID or unique short
prefix; omit it for the current impression. Check that the displayed results belong
to the intended run, especially after parameter/code edits.

## Logs

`celebi-cli log INDEX` reads a user command's combined stdout/stderr. The index is
zero-based (`celebi_user_step0.log`, etc.), not a task ID or a history offset.
Inspect later steps for failures after the first command. `log 0 --follow
--poll-interval 5` polls until interrupted; use a bounded session and monitor status.

Use `engine-logs` for workflow engine/orchestration logs and `engine-logs --fetch`
for a fresh backend fetch. `collect-logs` requests logs into Yuki storage. A lack of
user logs can mean the job never started; inspect engine logs and detailed status.

## Collect retained outputs

Collection copies files from a runner into **Yuki**, not the local task directory.

| Command | Selection |
|---------|-----------|
| `celebi-cli collect` | Plots and logs (light default) |
| `celebi-cli collect all` | All stageout outputs and logs |
| `celebi-cli collect plots` | Plot files |
| `celebi-cli collect data` | Data files |
| `celebi-cli collect logs` | Logs |
| `celebi-cli collect '*.root'` | Matching stageout files |
| `celebi-cli collect --allow-failed logs` | Available diagnostics from a failed workflow |
| `celebi-cli collect-outputs` | All stageout outputs |

Quote globs so the local shell does not expand them. Check skipped/failed entries;
a successful job or collection request does not prove every requested file arrived.
`--allow-failed` permits collection of files that already exist for a failed
workflow; it does not mark the workflow successful or guarantee complete stageout.
`refresh-filelists` refreshes runner listings, useful after terminal status froze a
listing; it does not fetch result bytes. `whereabouts` distinguishes Yuki storage
from runner caches. Do not infer missing results from absent local `stageout/`.

## Download and view

Get the actual page from `view-url`, inspect its links, and download the selected
file's resolved URL to a user-requested destination or temporary file. Do not assume
localhost, port 3315, or construct an undocumented file-view route. For example,
after resolving an actual link:

```bash
curl --fail --location "$file_url" --output "$destination"
```

Use the available image viewer for raster plots or suitable ROOT/Python tools for
scientific files. `celebi-cli imgcat` lists available images and `imgcat FILE` can
display them in a compatible terminal. Fetching a web page alone does not verify
that a downloaded artifact is valid; inspect the resulting file.

## Assess results and failures

Verify expected files, dimensions/entries, fit convergence, uncertainties, and
analysis-specific checks. Interpret chi-square with the degrees of freedom,
uncertainty model, and residuals; fixed chi-square/NDF thresholds do not diagnose
overfitting by themselves. Separate observed results from physical interpretation.

- Unknown `log` command: check installed CLI/version with `--help`; changing object
  directory does not add a missing CLI command.
- Empty page/404: check DITE connectivity, impression identity, and collection.
  `view` opens a browser; it does not start Yuki.
- Empty outputs: inspect job state, command logs, stageout path, selection filter,
  and data locations before rerunning.

Source anchors: Celebi `CelebiChrono/kernel/vtask_job.py`; Yuki
`Yuki/server/routes/status.py`, `routes/workflow.py`, `Yuki/kernel/jobs/`, and
`kernel/storage/file_types.py`.

## Check provenance and data completeness

When comparing runs, select the intended impression explicitly with `view-url ID`
and record its parameters alongside measured results. A view of the current task
may change after an edit. Use `celebi-impression-management` for historical snapshots
or archive exports when installed.

For unavailable bytes or a required cross-runner copy, use
`celebi-data-management`. For HTTP/worker faults use `yuki-service-management`:
`yuki server status` is currently a placeholder and does not prove the result
service is healthy. Preserve the observed failure instead of assuming a browser
error means the scientific task failed.
