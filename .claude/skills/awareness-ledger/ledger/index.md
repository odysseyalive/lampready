# Awareness Ledger Index

*Auto-generated. Last updated: 2026-07-24*

## By Tag

### mariadb
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — MariaDB start failed silently: mysqld_safe piped through `logger` with no syslog listener, then `mariadb.socket` held port 3306 (resolved)

### mysqld_safe
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — `syslog` in 50-mysqld_safe.cnf makes mariadbd die of SIGPIPE before it can log (resolved)

### syslog
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — `/dev/log` exists but journald never listens; `logger` exits 1 (resolved)

### systemd
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — container systemd (PID 1 as uid 1000) is wedged but still binds socket units (resolved)

### podman
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — `podman exec -d` discards stdout/stderr, hiding the real error (resolved)

### container-startup
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — `bash box up` aborts at "Waiting for MariaDB socket to become available" (resolved)

### debian-box
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — fixes in `start_mariadb()`, `_mariadb_config`, `_timeout` init, `session.gc_maxlifetime` sed (resolved)

### up-action
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — both startup branches affected; `init.lock` caused `import_db` to be skipped after the failure (resolved)

### diagnostics
- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — `mysqld_safe --dry-run` exiting 0 isolated the fault to the launch pipeline (resolved)

## By Status

### Active

*(No records yet)*

### Resolved

- INC-2026-07-24-mariadb-start-syslog-and-socket-unit — MariaDB failed to start under `bash box up`; two stacked causes: `mysqld_safe`'s syslog pipeline killing mariadbd via SIGPIPE, and systemd's `mariadb.socket` holding port 3306 [mariadb, mysqld_safe, syslog, systemd, podman, container-startup, debian-box, up-action, diagnostics]

### Under Review

*(No records yet)*

## Relationship Map

*(No cross-references yet)*

## Statistics

| Type | Total | Active | Resolved | Deprecated |
|------|-------|--------|----------|------------|
| Incidents | 1 | 0 | 1 | 0 |
| Decisions | 0 | 0 | 0 | 0 |
| Patterns | 0 | 0 | 0 | 0 |
| Flows | 0 | 0 | 0 | 0 |
