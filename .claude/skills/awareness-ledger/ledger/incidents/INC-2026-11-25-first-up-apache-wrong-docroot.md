# INC-2026-11-25-first-up-apache-wrong-docroot

**Status:** active
**Tags:** apache, systemd, container-startup, up-action, init, ssl, debian-box
**Related:** INC-2026-11-25-xdebug-develop-mode-cpu-burn, INC-2026-07-24-mariadb-start-syslog-and-socket-unit

## What Happened

On a fresh site's first `bash box up`, Apache served `/var/www/html`, so the
site returned 404. `apache2ctl -k start` reported "already running". The first
`apachectl graceful` (from `bash box xdebug`) killed Apache. The next
`xdebug` call started it again via podman exec, and from then on it served the
site correctly.

## Timeline

| Time/Commit | Event |
|-------------|-------|
| 2026-11-25 | Found during the throwaway-site test of 642bbd7. Not fixed |

## Root Cause

The container's own systemd starts Apache at boot, before `init` writes the
vhosts into `sites-enabled`. That Apache also can't read the snakeoil key, so
a graceful reload fails and Apache exits.

## Contributing Factors (Swiss Cheese Layers)

1. **Two start paths**: the container's systemd unit and box's `podman exec` start both try to own Apache.
2. **Key permissions**: the snakeoil key isn't readable by the mapped user.
3. **Later starts are fine**: after any later `up` or `rebuild`, box starts Apache itself, so the bug shows up only on a site's first `up`.

## Resolution

Not fixed. Workaround: after the first `up`, run `bash box xdebug off` (or `on`) once, or run `halt` then `up`.

## Lessons Learned

> **"The container's systemd starts Apache at boot, before init writes the vhosts."**

*— Captured 2026-11-25, source: implementation subagent test report*

## Prevention

Mask or disable `apache2.service` in the image, so that box is the only thing that starts Apache. Fix the snakeoil key's permissions.
