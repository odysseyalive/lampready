# INC-2026-11-25-xdebug-develop-mode-cpu-burn

**Status:** resolved
**Tags:** xdebug, php, cpu, performance, podman, systemd, debian-box, up-action, rebuild-action, destroy-action, docs
**Related:** INC-2026-11-25-http-port-publish-mismatch, INC-2026-11-25-first-up-apache-wrong-docroot, INC-2026-11-25-halt-unclean-mariadb-shutdown

## What Happened

realamish.com (`~/www/amishwbf.com`) used 136% CPU (about 1.4-2 of the 4 cores,
measured over 30 s) while ra-dev's Playwright suite sent about 1,100 requests
per minute. `xdebug.log` logged 14,643 "could not connect" attempts and reached
12 MB. Every PHP request ran Xdebug's develop mode and tried to reach an IDE on
`host.containers.internal:9003`, where nothing was listening.

## Timeline

| Time/Commit | Event |
|-------------|-------|
| 2026-11-25 14:39 | cpu-dev, with Francis's approval, edits the container's `xdebug.ini` to debug+trigger and runs `podman update --cpus=2`. CPU drops to 39% at the same load |
| 2026-11-25 22:42Z | cpu-dev asks lampready-dev what default lampready should ship |
| 2026-11-25 | lampready-dev finds the live fix won't survive a restart, because units use `podman generate systemd --new` |
| 642bbd7 | Fix: Xdebug opt-in and toggleable, CPU limit, data-safe `rebuild`. Tested on a throwaway site, pushed |
| 2026-11-25 23:07Z | Francis approves rollout to the 6 sites that have `update`. cpu-dev's helper does it one site at a time |

## Root Cause

`_xdebug_config` in `debian-box` unconditionally wrote
`xdebug.mode=develop,debug` and `xdebug.start_with_request=yes` (with the
default log level 7) for every site. The `XDEBUG_ENABLE="0"` header setting
existed but nothing read it.

## Contributing Factors (Swiss Cheese Layers)

1. **Unused setting**: `XDEBUG_ENABLE` looked like it controlled Xdebug, so nobody suspected Xdebug was on.
2. **No trigger gating**: `start_with_request=yes` made every request try to reach the IDE, including test traffic.
3. **Verbose logging**: log level 7 wrote a line for every failed connection.
4. **No CPU limit**: one busy container could take the whole machine.
5. **Ephemeral containers**: `podman generate systemd --new` recreates the container from the image on every start, so live fixes inside it are lost.
6. **Copied boxes**: each site carries its own copy of `box` from a different vintage, so a fix in the repo reaches no site until it is updated.

## Resolution

Commit 642bbd7:
- `XDEBUG_ENABLE` defaults to `"0"` and gives `xdebug.mode=off`. When on, it uses `XDEBUG_MODE` (default `debug`) with `start_with_request=trigger` and `log_level=1`.
- `bash box xdebug on|off|status` rewrites the host file `.container/php/99-xdebug-local.ini`, which is bind-mounted into `conf.d`, then reloads Apache. No rebuild needed, and the setting survives container recreation.
- `CONTAINER_CPUS="2"` is passed as `--cpus` to `podman create`. If the rootless `cpu` cgroup controller isn't delegated, box warns and skips the limit.
- New `bash box rebuild`: shuts MariaDB down cleanly, rebuilds the image and the unit, and keeps `.container` data. It requires `init.lock`.
- Docs: `destroy` deletes all of `.container`, including DB data and `sites-enabled`. CLAUDE.md and README had claimed otherwise. README had also told users to run `bash box reset` (which wipes everything) before a rebuild.

## Lessons Learned

> **"Every request then runs develop mode and tries to reach an IDE on host.containers.internal:9003."**

*— Captured 2026-11-25, source: cpu-dev's diagnosis, conversation*

> **The live fix will not last: every service start creates a fresh container from the image.**

*— Captured 2026-11-25, source: lampready-dev, conversation*

## Prevention

- Anything that has to persist goes into the image or into a bind-mounted file under `.container/`, never a live edit in the container.
- Rollout: a site gets the fix only after `bash box update` and `bash box rebuild` (the rebuild is needed once). Sites with no `.container` only need `update`, because rebuild refuses without `init.lock`.
- Old boxes without `update` (nevadacl, the Ubuntu-focal boxes, and the boxes with no delimiter) still carry develop-mode Xdebug until they are migrated.
