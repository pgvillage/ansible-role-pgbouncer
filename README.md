PgBouncer
=========

PGBouncer is a connection pooler for PostgreSQL.
This role installs pgbouncer as a systemd service and configures it to pool the normal PostgreSQL port, or the solon-proxy port.
This role is part of PgVillage, which is an opinated PostgreSQL deployment for Virtual Machines.

Requirements
------------

None

Role Variables
--------------

Please see [defaults](https://github.com/pgvillage/ansible-role-pgbouncer/blob/main/defaults/main.yml) for all variables


Dependencies
------------

No dependencies


Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - pgvillage.pgbouncer

Multiple instances
------------------

Every entry in `pgbouncer_instances` becomes a separate pgbouncer process (systemd unit `pgbouncer@<name>`,
config `/etc/pgbouncer/pgbouncer-<name>.ini`). Values not set on an instance are taken from
`pgbouncer_instance_defaults`. Each instance needs a unique `port`.
This allows pooling multiple PostgreSQL clusters, or one cluster with different pool settings:

    - hosts: servers
      vars:
        pgbouncer_instances:
          pg15:
            pgdata: /var/lib/pgsql/15/data
            pgport: 5432
            port: 6432
          pg16:
            pgdata: /var/lib/pgsql/16/data
            pgport: 5433
            port: 6433
          pg16_session:
            pgdata: /var/lib/pgsql/16/data
            pgport: 5433
            port: 6434
            pool_mode: session
      roles:
         - pgvillage.pgbouncer

License
-------

PostgreSQL

Author Information
------------------

PgVillage is an Open Community.
Main contributor is Nibble-IT.
