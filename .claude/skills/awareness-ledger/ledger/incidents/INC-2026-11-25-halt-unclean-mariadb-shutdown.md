# INC-2026-11-25-halt-unclean-mariadb-shutdown

**Status:** active
**Tags:** mariadb, halt-action, podman, systemd, data-integrity, debian-box
**Related:** INC-2026-11-25-xdebug-develop-mode-cpu-burn, INC-2026-07-24-mariadb-start-syslog-and-socket-unit

## What Happened

`bash box halt` stops the systemd service and the container without shutting
MariaDB down first. podman kills the container after its stop timeout, so
MariaDB never gets a clean shutdown and has to do crash recovery on the next
start.

## Timeline

| Time/Commit | Event |
|-------------|-------|
| 642bbd7 | `rebuild` gets a clean `mysqladmin shutdown`. `halt` is unchanged |
| 2026-11-25 | Noted during testing. Not fixed |

## Root Cause

The `halt` action calls `systemctl --user stop` and `podman stop` with no
`mysqladmin shutdown` beforehand. MariaDB is started with `podman exec`
rather than as a container service, so stopping the container doesn't stop it
gracefully.

## Contributing Factors (Swiss Cheese Layers)

1. **MariaDB runs outside the container's init**: nothing inside the container signals it at stop time.
2. **InnoDB recovery masks it**: data usually survives, so the problem doesn't show.

## Resolution

Not fixed. Proposed: reuse the `mysqladmin shutdown` step from `rebuild` in `halt` and `destroy`.

## Lessons Learned

> **"halt kills MariaDB without a clean shutdown; podman kills the container after its stop timeout. Only rebuild now does a clean mysqladmin shutdown."**

*— Captured 2026-11-25, source: implementation subagent test report*

## Prevention

Any action that stops the container should shut MariaDB down first.
