---
name: yuki-service-management
description: Set up or diagnose the Yuki Flask/Celery service and Docker deployment used by Celebi, including connectivity, broker health, storage roots, and process lifecycle limitations.
---

# Operate the Yuki service

Use this skill for service deployment or service-level faults. For a single failed
analysis, start with task and engine logs; restarting Yuki is not the default fix.
For runner registration use `celebi-runner-management` when installed.

## Establish the deployment

Determine whether Yuki is a native process, a managed container, or a Compose dev
service. Record the actual client DITE endpoint, service host/container, persistent
storage mount, process owner, and version of both CelebiChrono and Yuki. Reuse the
existing lifecycle mechanism.

Yuki uses Flask plus Celery, with RabbitMQ at `amqp://localhost` in the checked
configuration. Native `yuki server start` starts Flask on `0.0.0.0:3315` and a Celery
worker, ensures a native runner named `local` exists, then waits on both processes.
An existing `local` definition is preserved. It runs in the foreground; arrange process
management according to the deployment rather than assuming daemonization.

The checked `yuki server status` implementation returns a placeholder `running`.
It is **not a health check**. `server stop` reads daemon PID files that `server start`
does not create in the inspected implementation. Verify actual process ownership
and the service manager/container before relying on those commands for lifecycle.

## Select an installation/startup path

For an authorized native source installation, install into the intended environment
with `pip install -e .` from the Yuki checkout. Make the matching CelebiChrono version
available there and provide RabbitMQ before starting the service. The standard
package now includes Snakemake for native execution and expects the host environment
to provide the `conda` command used for project environments.

For an authorized container deployment, inspect supported options first:

```bash
yuki docker run --help
yuki docker restart --help
```

`yuki docker run IMAGE --yuki-dir PATH --port PORT` mounts persistent host storage
and maps PORT to container port 3315. Use actual desired values and inspect existing
containers/ports before creating another instance. The all-in-one image starts its
own RabbitMQ. `yuki docker restart [CONTAINER]` acts on an existing container; inspect
the selection behavior before omitting its name on a shared host.

For development from the checkout, Compose mounts the Yuki source and optional
CelebiChrono source (`CELEBI_DIR`) for local development. The checked Compose file
targets macOS Docker Desktop; use its documented Linux alternative where applicable.
Do not replace an existing production deployment with the dev Compose setup merely
to test a code edit.

## Diagnose by component

1. From the analysis client, `celebi-cli dite` tests the configured connection.
   `set-dite URL` changes that client's endpoint; it does not start the service.
2. Inspect the real service/container status, HTTP response at the configured
   endpoint, and Flask logs. Use `docker logs CONTAINER` for the selected container
   when applicable. An HTTP response establishes reachability, not worker health.
3. For queued work, inspect Celery worker and RabbitMQ connectivity/logs. A healthy
   web server can coexist with a stopped worker or unavailable broker.
4. For backend execution, inspect `celebi-cli runners`, `test-runner NAME`, and
   task `engine-logs --fetch`. Separate SSH/REANA failures from Yuki availability.
   A registered `local` native runner also needs a live `yuki native-runner start`
   host agent; registration is not a liveness check.

## Storage-root caveat

Many current components consult `YUKIDIR`, falling back to `~/.Yuki`, but the checked
`Yuki/server/config.py` still constructs paths directly under process HOME.
Other helpers use `os.environ.get` without normalizing an empty value. Do not assume
all routes honor a custom or empty `YUKIDIR` consistently.

For relocated storage, set a non-empty intended path and inspect the exact route/
worker resolving it, plus container mounts and effective process HOME. Workflow
metadata, runner registry, collected results, and logs must refer to the same
intended deployment. A missing file in another storage root is not evidence that
results were deleted. Do not move storage or rewrite runner config until the
mismatch and desired layout are established.

After a service change, verify real HTTP connectivity, worker/broker health, runner
visibility, and access to a known impression. Execute a small workflow only when
execution is in scope; do not claim full recovery from placeholder status output.

Bookkeeping deployments additionally expose scoped result relay and runner support
through `Yuki/services/celebi_server_booker.py` and
`Yuki/server/routes/server_booking.py`; this does not turn the Yuki service itself
into the public analysis website.

Source anchors: Yuki `Yuki/server_main.py`, `Yuki/server/app.py`, `config.py`,
`tasks.py` under the server package, `Yuki/main.py`, and `docker-compose.yml`.
