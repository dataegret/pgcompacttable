pgcompacttable is a tool for reducing size of bloated tables and indexes without heavy locks.

It is designed to reorganize data in tables and rebuild indexes in order to revert back disk space without database performance impact.

Setup
--------------------
`pgcompacttable` is written in Perl and requires Perl DBI library, obviously with PostgreSQL support module. Dependencies can be easy installed:
* on Debian-based Linux OS with `apt-get install libdbi-perl libdbd-pg-perl`
* RedHat/Centos with `yum install perl-Time-HiRes perl-DBI perl-DBD-Pg`

In target database contrib module [`pgstattuple`](https://www.postgresql.org/docs/current/static/pgstattuple.html) should be installed via `create extension if not exists pgstattuple;`

Run
--------------------
`pgcompacttable` can be run from any OS user (and even on another host, see `--host` option; although it is recommended to run it on same host with target database), but PostgreSQL Superuser access is required. Preferred way is to run as PostgreSQL cluster owner, usually `postgres`. In this case `pgcompacttable` can perform `ionice -c 3` for PostgreSQL backend pid to lower IO priority.

`pgcompacttable --man`
Prints list of available options and usage information.

`pgcompacttable --all --verbose`
Compacts all bloated tables in all databases in the cluster, including indexes. Prints additional progress information.

`pgcompacttable --dbname billing --exclude-schema pgq`
Compacts all bloated tables in the billing database and their bloated indexes excepts ones that are in the pgq schema.

`pgcompacttable --dbname billing -t operations -f`
Force compact table operations in database billing.
Notice, tables and indexes with bloat less than 20% (hardcoded MINIMAL_COMPACT_PERCENT constant) are considered normal and not processed until option `--force` is taken.

Compatibility
--------------------
pgcompacttable currently supports PostgreSQL starting from 9.2.
Rebuilds partial indexes, functional, unique, used for foreign keys and indexes already placed on another tablespace are supported.

What about pg_repack?
--------------------
Unlike another popular tool [pg_repack](https://github.com/reorg/pg_repack) this tool has some advantages:
 * does not requires lots of free space
   * tables are processed in-place
   * indexes are rebuild one by one, from smallest to largest
   therefore maximum space required is the size of the largest index
 * tables are processed with adaptive delays to prevent heavy IO and replication lag spikes (see `--delay-ratio` option)
