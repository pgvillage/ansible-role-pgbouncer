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

License
-------

PostgreSQL

Author Information
------------------

PgVillage is an Open Community.
Main contributor is Nibble-IT.
