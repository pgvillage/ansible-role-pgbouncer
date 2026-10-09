# pgvillage.pgbouncer – Role API

This document describes all variables that can be set for the `pgvillage.pgbouncer` role.
Defaults are defined in [defaults/main.yml](../defaults/main.yml).

## Global variables

| Variable | Default | Description |
|---|---|---|
| `pgbouncer_install_dir` | `/usr/bin` | Directory containing the pgbouncer binary, used in `ExecStart` of the `pgbouncer@.service` systemd unit. |
| `pgbouncer_os_user` | `pgbouncer` | OS user that runs pgbouncer and owns its config, log and run files. Created as a system account. Also mapped to `auth_user` in `pg_ident.conf` (see `manage_ident`). |
| `pgbouncer_os_group` | `pgbouncer` | OS group for `pgbouncer_os_user`. Created as a system group. |
| `pgbouncer_disable_default_service` | `true` | Stop and disable the `pgbouncer.service` shipped by the distribution package, so it does not compete with the `pgbouncer@<instance>` units for ports. |
| `pgbouncer_instances` | `{main: {}}` | Dictionary of pgbouncer instances to deploy. See [Instances](#instances). |
| `pgbouncer_instance_defaults` | see below | Default settings merged (recursively) into every instance. See [Instance settings](#instance-settings). |

## Instances

One pgbouncer process is deployed per entry in `pgbouncer_instances`:

- systemd unit: `pgbouncer@<name>`
- config file: `/etc/pgbouncer/pgbouncer-<name>.ini`
- log file: `/var/log/pgbouncer/pgbouncer-<name>.log`
- pid file: `/var/run/pgbouncer/pgbouncer-<name>.pid`

Rules:

- `pgbouncer_instances` must not be empty.
- Instance names may only contain `[A-Za-z0-9_-]`.
- Every instance needs a unique `port`.

Every entry is combined with `pgbouncer_instance_defaults`, so only the values that differ need to be set.

```yaml
pgbouncer_instances:
  pg15:
    pgdata: /var/lib/pgsql/15/data
    pgport: 5432
    port: 6432
  pg16:
    pgdata: /var/lib/pgsql/16/data
    pgport: 5433
    port: 6433
```

## Instance settings

These keys can be set in `pgbouncer_instance_defaults` (for all instances) or per instance in `pgbouncer_instances`.

### PostgreSQL integration

| Key | Default | Description |
|---|---|---|
| `manage_hba` | `true` | Add `pg_hba.conf` entries: peer auth (`map=pgbouncer`) for `auth_user` on `auth_db`, and `scram-sha-256` for all other local connections. PostgreSQL is reloaded on change. |
| `manage_ident` | `true` | Add a `pgbouncer <pgbouncer_os_user> <auth_user>` mapping to `pg_ident.conf`, so pgbouncer can log in as `auth_user` without a password. |
| `pgdata` | `/var/lib/postgresql/data` | Data directory of the pooled PostgreSQL cluster, used to locate `pg_hba.conf` and `pg_ident.conf`. |
| `pghost` | `/tmp` | Host or unix socket directory pgbouncer connects to. |
| `pgport` | `5432` | Port of the pooled PostgreSQL cluster. Used by pgbouncer and to create the auth db / user. |
| `pgenv` | `{service: master}` | libpq environment used to connect to PostgreSQL when creating the auth db, user and lookup function. Every key becomes a `PG<KEY>` environment variable, e.g. `service` → `PGSERVICE`, `host` → `PGHOST`. |

### Authentication

| Key | Default | Description |
|---|---|---|
| `auth_user` | `pgbouncer_auth_user` | PostgreSQL user that pgbouncer uses to look up passwords (`auth_user` in pgbouncer.ini). The role creates it and grants it `EXECUTE` on `public.lookup()`. |
| `auth_db` | `pgbouncer_auth_db` | Database in which the `public.lookup()` function is created (`auth_dbname` in pgbouncer.ini). Must not be `pgbouncer`, as that name is reserved for the pgbouncer admin console. |
| `auth_type` | `scram-sha-256` | How pgbouncer authenticates clients (`any`, `trust`, `plain`, `md5`, `scram-sha-256`, `cert`, `hba`, `pam`). |

### Listener and pooling

| Key | Default | Description |
|---|---|---|
| `listen_addr` | `*` | Address(es) to listen on for TCP connections, `*` for all. |
| `port` | `6432` | Port to listen on. Must be unique per instance. |
| `pool_mode` | `transaction` | When a server connection is released back to the pool (`session`, `transaction`, `statement`). |
| `max_client_conn` | `100` | Maximum number of client connections allowed. |
| `default_pool_size` | `20` | Number of server connections per user/database pair. |
| `min_pool_size` | `0` | Minimum number of server connections to keep in the pool. |

### TLS

| Key | Default | Description |
|---|---|---|
| `server_cert_file` | `""` | Certificate pgbouncer presents to clients (`client_tls_cert_file`). Client TLS is only enabled when this is set. |
| `server_key_file` | `""` | Private key for `server_cert_file` (`client_tls_key_file`). |
| `ca_file` | `""` | CA file used to verify the PostgreSQL server certificate (`server_tls_ca_file`). |
| `client_cert_file` | `""` | Certificate pgbouncer presents to PostgreSQL (`server_tls_cert_file`). Only needed when PostgreSQL requires client certificates; enabled when this is set. |
| `client_key_file` | `""` | Private key for `client_cert_file` (`server_tls_key_file`). |

Connections from pgbouncer to PostgreSQL always use `server_tls_sslmode = require`.
