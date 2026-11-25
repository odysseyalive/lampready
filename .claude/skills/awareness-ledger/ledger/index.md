# Awareness Ledger Index

*Auto-generated. Last updated: 2026-11-25*

## By Tag

### mariadb
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — MariaDB start failed silently: mysqld_safe piped through `logger` with no syslog listener, then `mariadb.socket` held port 3306 (resolved)
- INC-2026-11-25-halt-unclean-mariadb-shutdown — `halt` stops the container without `mysqladmin shutdown`; only `rebuild` shuts down cleanly (active)

### mysqld_safe
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — `syslog` in 50-mysqld_safe.cnf makes mariadbd die of SIGPIPE before it can log (resolved)

### syslog
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — `/dev/log` exists but journald never listens; `logger` exits 1 (resolved)

### systemd
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — container systemd (PID 1 as uid 1000) is wedged but still binds socket units (resolved)
- INC-2026-11-25-xdebug-develop-mode-cpu-burn — `podman generate systemd --new` recreates the container on every start, so live edits inside it are lost (resolved)
- INC-2026-11-25-first-up-apache-wrong-docroot — container systemd starts Apache before `init` writes the vhosts (active)
- INC-2026-11-25-halt-unclean-mariadb-shutdown — `systemctl --user stop` gives MariaDB no clean shutdown (active)

### podman
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — `podman exec -d` discards stdout/stderr, hiding the real error (resolved)
- INC-2026-11-25-xdebug-develop-mode-cpu-burn — `--cpus` must be set at `podman create`; `podman update` is lost on restart (resolved)
- INC-2026-11-25-http-port-publish-mismatch — publishes `HTTP_PORT:80` while Apache listens on `HTTP_PORT` (active)
- INC-2026-11-25-halt-unclean-mariadb-shutdown — podman kills the container after its stop timeout (active)

### container-startup
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — `bash box up` aborts at "Waiting for MariaDB socket to become available" (resolved)
- INC-2026-11-25-first-up-apache-wrong-docroot — first `up` serves `/var/www/html`; first graceful kills Apache (active)

### debian-box
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — fixes in `start_mariadb()`, `_mariadb_config`, `_timeout` init, `session.gc_maxlifetime` sed (resolved)
- INC-2026-11-25-xdebug-develop-mode-cpu-burn — `_xdebug_config` forced develop,debug + start_with_request=yes; XDEBUG_ENABLE unused (resolved, 642bbd7)
- INC-2026-11-25-http-port-publish-mismatch — `setup_systemd_service` publish mapping (active)
- INC-2026-11-25-first-up-apache-wrong-docroot — two owners of the Apache start (active)
- INC-2026-11-25-halt-unclean-mariadb-shutdown — `halt` action (active)

### up-action
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — both startup branches affected; `init.lock` caused `import_db` to be skipped after the failure (resolved)
- INC-2026-11-25-first-up-apache-wrong-docroot — affects only a site's first `up` (active)

### diagnostics
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — `mysqld_safe --dry-run` exiting 0 isolated the fault to the launch pipeline (resolved)

### xdebug
- INC-2026-11-25-xdebug-develop-mode-cpu-burn — 136% CPU and a 12 MB xdebug.log under Playwright load; now off by default, toggled with `bash box xdebug` (resolved)

### php
- INC-2026-11-25-xdebug-develop-mode-cpu-burn — override file `.container/php/99-xdebug-local.ini` bind-mounted into conf.d (resolved)

### cpu
- INC-2026-11-25-xdebug-develop-mode-cpu-burn — `CONTAINER_CPUS="2"` default; warns when the cgroup cpu controller isn't delegated (resolved)

### performance
- INC-2026-11-25-xdebug-develop-mode-cpu-burn — develop mode + start_with_request=yes on every request (resolved)

### rebuild-action
- INC-2026-11-25-xdebug-develop-mode-cpu-burn — data-safe `rebuild` added; it requires `init.lock` (resolved)

### destroy-action
- INC-2026-11-25-xdebug-develop-mode-cpu-burn — `destroy` deletes all of `.container`, DB data included; docs corrected (resolved)

### docs
- INC-2026-11-25-xdebug-develop-mode-cpu-burn — CLAUDE.md/README claimed destroy kept data; README told users to `reset` before a rebuild (resolved)

### apache
- INC-2026-11-25-http-port-publish-mismatch — Apache listens on `HTTP_PORT` inside the container (active)
- INC-2026-11-25-first-up-apache-wrong-docroot — snakeoil key unreadable, so graceful fails (active)

### ports
- INC-2026-11-25-http-port-publish-mismatch — non-default `HTTP_PORT` unreachable from the host (active)

### networking
- INC-2026-11-25-http-port-publish-mismatch — publish target port mismatch (active)

### setup_systemd_service
- INC-2026-11-25-http-port-publish-mismatch — `--publish "${HTTP_PORT}:80"` (active)

### init
- INC-2026-11-25-first-up-apache-wrong-docroot — vhosts are written after Apache has already started (active)

### ssl
- INC-2026-11-25-first-up-apache-wrong-docroot — snakeoil key permissions (active)

### halt-action
- INC-2026-11-25-halt-unclean-mariadb-shutdown — no `mysqladmin shutdown` before stop (active)

### data-integrity
- INC-2026-11-25-halt-unclean-mariadb-shutdown — InnoDB crash recovery on every restart after halt (active)

## By Status

### Active

- INC-2026-11-25-http-port-publish-mismatch — non-default `HTTP_PORT` is unreachable from the host: podman publishes `HTTP_PORT:80`, but Apache listens on `HTTP_PORT` [podman, apache, ports, networking, debian-box, setup_systemd_service]
- INC-2026-11-25-first-up-apache-wrong-docroot — on a site's first `up`, Apache started by the container's systemd serves `/var/www/html`, and the first graceful kills it (workaround: `bash box xdebug off` once) [apache, systemd, container-startup, up-action, init, ssl, debian-box]
- INC-2026-11-25-halt-unclean-mariadb-shutdown — `halt` stops the container without shutting MariaDB down cleanly [mariadb, halt-action, podman, systemd, data-integrity, debian-box]

### Resolved

- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — MariaDB failed to start under `bash box up`; two stacked causes: `mysqld_safe`'s syslog pipeline killing mariadbd via SIGPIPE, and systemd's `mariadb.socket` holding port 3306 [mariadb, mysqld_safe, syslog, systemd, podman, container-startup, debian-box, up-action, diagnostics]
- INC-2026-11-25-xdebug-develop-mode-cpu-burn — Xdebug ran develop mode on every request (136% CPU, 12 MB log); fixed in 642bbd7 with off-by-default Xdebug, `bash box xdebug`, `CONTAINER_CPUS`, and a data-safe `rebuild` [xdebug, php, cpu, performance, podman, systemd, debian-box, up-action, rebuild-action, destroy-action, docs]

### Under Review

*(No records yet)*

## Relationship Map

- INC-2026-11-25-xdebug-develop-mode-cpu-burn -> related to INC-2026-11-25-http-port-publish-mismatch, INC-2026-11-25-first-up-apache-wrong-docroot, INC-2026-11-25-halt-unclean-mariadb-shutdown
- INC-2026-11-25-first-up-apache-wrong-docroot -> related to INC-2026-11-25-xdebug-develop-mode-cpu-burn, INC-2026-07-24-mariadb-start-syslog-and-socket-unit
- INC-2026-11-25-halt-unclean-mariadb-shutdown -> related to INC-2026-11-25-xdebug-develop-mode-cpu-burn, INC-2026-07-24-mariadb-start-syslog-and-socket-unit
- INC-2026-11-25-http-port-publish-mismatch -> related to INC-2026-11-25-xdebug-develop-mode-cpu-burn

## Statistics

| Type | Total | Active | Resolved | Deprecated |
|------|-------|--------|----------|------------|
| Incidents | 5 | 3 | 2 | 0 |
| Decisions | 0 | 0 | 0 | 0 |
| Patterns | 0 | 0 | 0 | 0 |
| Flows | 0 | 0 | 0 | 0 |
