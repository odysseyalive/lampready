# INC-2026-07-24-mariadb-start-syslog-and-socket-unit

**Status:** resolved
**Tags:** mariadb, mysqld_safe, syslog, systemd, podman, container-startup, debian-box, up-action, diagnostics
**Related:** —

## What Happened

`bash box up` for an affected site built the image successfully, then
aborted at:

```
Waiting for MariaDB socket to become available...
❌ MariaDB manual start failed.
```

The diagnostic tail the script prints showed only benign warnings (`io_uring …
ENOSYS`, `innodb-large-prefix was removed`, `--log-bin`), none of which were
related to the failure. `/var/lib/mysql/mysql.sock` never appeared and no
`mariadbd` process was running.

Two independent defects were present. The first prevented the second from ever
being observed.

## Timeline

| Time/Commit | Event |
|-------------|-------|
| 12:56 | `bash box up` runs first-time init; `mysql_install_db` succeeds, datadir populated |
| 12:56 | `mysqld_safe` launched via `podman exec -d`; socket never appears; script exits 1 |
| 12:56 | In-container `init` had already created `.container/init.lock`, so subsequent `up` runs skip `import_db` |
| 14:19 | Manual `/usr/sbin/mariadbd` run proves engine + config are sound (InnoDB inits, buffer pool loads) |
| 14:20 | `mysqld_safe --dry-run` exits 0 — every pre-flight check passes, isolating the fault to the launch pipeline |
| 14:2x | `echo hi \| logger` in-container returns `logger: socket /dev/log: Connection refused`, rc=1 — root cause 1 confirmed |
| 14:38 | Fix 1 applied; `mariadbd` now launches and logs, revealing `Bind on TCP/IP port. Got error: 98` — root cause 2 |
| 14:38 | `ss -lntp` shows `systemd` (PID 1) holding `*:3306` via the enabled `mariadb.socket` unit |
| 14:45 | Fix 2 applied, image rebuilt; `ready for connections`, Apache on 80/443, site returns 200 |
| 00cba7b | Both fixes committed |

## Root Cause

**1. syslog dependency in the MariaDB startup wrapper.** Debian ships
`/etc/mysql/mariadb.conf.d/50-mysqld_safe.cnf` with `syslog` under
`[mysqld_safe]`. With that set, `mysqld_safe` does not exec `mariadbd` directly —
it runs `mariadbd … 2>&1 | logger -t mysqld -p daemon.error`. The container's
`logger` exits 1 immediately (`socket /dev/log: Connection refused`), so
`mariadbd` takes SIGPIPE on its first stderr write and dies **before** it opens
`log_error`. Result: zero lines in `mariadb.log`, no socket, wait loop times out.

**2. `mariadb.socket` holding port 3306.** The same container's systemd had the
`mariadb.socket` unit enabled and had bound `*:3306` for socket activation it
could never complete. Once defect 1 was fixed, `mariadbd` got far enough to fail
with `Can't start server: Bind on TCP/IP port. Got error: 98: Address already in
use`.

## Contributing Factors (Swiss Cheese Layers)

1. **Container systemd is half-alive** — PID 1 is `/lib/systemd/systemd` running
   as uid 1000 (`USER webuser` in the Dockerfile + `--user`/`--userns=keep-id`).
   It gets far enough to create `/dev/log` and bind socket units, but never
   completes boot: `systemctl is-system-running` returns `Failed to query system
   state: Connection timed out`, journald never starts, `podman logs` is empty.
   So the syslog socket exists but nothing listens, and 3306 is held by a service
   that will never start.
2. **`podman exec -d` discards stdout/stderr** — `mysqld_safe`'s own error
   output went nowhere, so the one process that knew what happened was silenced.
3. **`skip_log_error` + `syslog` in the distro config** — together they route all
   startup output away from the file the script tails on failure.
4. **The failure message pointed at the wrong log** — `mariadb.log` contained
   only `mysql_install_db`'s bootstrap warnings from minutes earlier. Those
   warnings look plausible enough (`io_uring … check seccomp filters`) to send
   diagnosis toward the container runtime instead of the startup wrapper.
5. **Two defects stacked** — fixing only the first would have surfaced a fresh,
   unrelated-looking error, which reads as "the fix didn't work."

## Resolution

Committed in `00cba7b`:

- `start_mariadb()` in `debian-box` launches with
  `--skip-syslog --log-error=/var/log/apache2/mariadb.log`, taking `mysqld_safe`'s
  `logging=file` branch so `logger` is never involved.
- Wrapper output is redirected to `/var/log/apache2/mysqld_safe.out`, and
  `mariadb_start_failure()` prints it alongside `mariadb.log` on failure.
- `_mariadb_config` masks the socket unit (`ln -sf /dev/null
  /etc/systemd/system/mariadb.socket`, plus removing the
  `sockets.target.wants` symlink) and comments out `syslog` / `skip_log_error`
  in `50-mysqld_safe.cnf`, so the image is safe even without the CLI flags.

Two unrelated bugs found in the same code path and fixed in the same commit:

- `_timeout` was never initialised before the wait loop on the
  already-initialised branch (`debian-box:826`), so that loop exited on its first
  iteration and never actually waited.
- An unterminated `sed` expression meant `session.gc_maxlifetime` was silently
  never applied to `php.ini`.

Recovery for the affected site required `bash box import` to be run manually:
init died at STEP 2, before `import_db`, but `.container/init.lock` already
existed, so later `up` runs skipped database creation.

## Lessons Learned

> **"The failure was undiagnosable because the wrapper's stderr was discarded by
> `podman exec -d` and mariadbd died before it could open `log_error`. The log
> tail the script printed showed only unrelated `mysql_install_db` bootstrap
> warnings, which sent diagnosis down the wrong path. `mysqld_safe --dry-run`
> exiting 0 was the turning point — it proved every pre-flight check passed and
> isolated the failure to the launch pipeline itself."**

*— Captured 2026-07-24, source: conversation*

> **"When a fix reveals a new error, that is usually progress, not a failed fix.
> The first defect was masking the second; both were present from the start."**

*— Captured 2026-07-24, source: conversation*

## Prevention

- Never start a service in this container through a path that depends on syslog,
  journald, or `systemctl`. The container's systemd is non-functional; every
  service is started manually via `podman exec`. Any future code that assumes a
  working `systemctl` or journal inside the container will fail the same way.
- Any background `podman exec -d` should redirect stdout/stderr to a file under
  `/var/log/apache2/` and that file should be printed on failure. A silent
  wrapper turns a five-minute diagnosis into an hour.
- When a startup check times out, verify what is holding the port
  (`ss -lntp` inside the container) before assuming the service itself is at
  fault — systemd socket units bind ports without running anything.

### Related hazards found but NOT fixed

- **`bash box destroy` deletes the database.** It runs `rm -rf .container`, which
  includes `.container/mariadb` (the `/var/lib/mysql` bind mount) and
  `.container/log`. `CLAUDE.md` states destroy preserves data. To rebuild an
  image without data loss, remove the image (`podman rmi
  localhost/<site>:private`) and re-run `bash box up` instead.
- **Container systemd is non-functional by accident, not design.** It is
  tolerated because every service is started manually. Worth an explicit decision
  on whether to keep systemd as PID 1 at all.
- **`tmp_table_size=256MB` / `max_heap_table_size=256MB`** in `_mariadb_config`
  use a non-standard `MB` suffix (MariaDB expects `K`/`M`/`G`). Currently
  accepted; worth normalising to `256M`.
