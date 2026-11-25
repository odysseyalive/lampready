# INC-2026-11-25-http-port-publish-mismatch

**Status:** active
**Tags:** podman, apache, ports, networking, debian-box, setup_systemd_service
**Related:** INC-2026-11-25-xdebug-develop-mode-cpu-burn

## What Happened

While testing 642bbd7 on a throwaway site with `HTTP_PORT=18089`, curl from the
host to port 18089 failed. Requests worked only from inside the container.

## Timeline

| Time/Commit | Event |
|-------------|-------|
| 2026-11-25 | Found during the throwaway-site test of 642bbd7. Not fixed |

## Root Cause

`podman create` publishes `${HTTP_PORT}:80` (and `${SSL_PORT}:443`), but
Apache inside the container is configured to listen on `HTTP_PORT` itself. With
a non-default port, the host forwards to container port 80, where nothing is
listening.

## Contributing Factors (Swiss Cheese Layers)

1. **Default ports hide it**: with `HTTP_PORT=80`, both sides agree.

## Resolution

None yet. Either publish `${HTTP_PORT}:${HTTP_PORT}`, or have Apache always listen on 80/443 inside the container.

## Lessons Learned

> **"podman publishes HTTP_PORT:80 but Apache inside listens on HTTP_PORT, so host curl on 18089 failed."**

*— Captured 2026-11-25, source: implementation subagent test report*

## Prevention

Test at least one non-default port whenever the publish or Listen config changes.
