# Book a project catalog to REANA

Use booking when the user wants to publish a Celebi project file catalog through
Yuki. It creates remote state and uploads selected content; it is not an alias for
`submit RUNNER` and is not a necessary step for every REANA analysis.

From the intended project, inspect `celebi-cli booking-server`. Existing stored
booking credentials may suffice. `celebi-cli register-booking-server --server URL`
can use the existing `REANA_ACCESS_TOKEN` environment variable; credentials are
stored on Yuki. Avoid printing tokens or adding them to project YAML/source.
Command-line `--server`/`--token` on booking override stored credentials when needed.

```bash
celebi-cli book-reana --path /actual/project --upload 'plots+logs'
```

Use the actual project path. The upload selector defaults to `plots+logs`; supported
choices include `plots`, `data`, `all`, or a quoted glob. Choose based on the intended
catalog and available results; do not upload all data just to simplify selection.
`--stageout` is a deprecated alias for `--upload all`. `--no-stream` waits for complete
output instead of streaming. Keep TLS verification enabled unless a specific,
understood deployment requirement calls for `--insecure`.

Inspect streaming/final messages and the returned catalog/workflow identity. Check
that selected data were available and published. If the client disconnects, inspect
server/backend state before creating another booking. Record what was actually
published and any skipped/failed items rather than inferring completion from the
command's exit status alone.

Source anchors: Celebi `CelebiChrono/celebi_cli/commands/booking.py`,
`CelebiChrono/interface/shell_modules/reana_booking.py`; Yuki
`Yuki/kernel/reana_booker.py` and `Yuki/server/routes/booking.py`.
