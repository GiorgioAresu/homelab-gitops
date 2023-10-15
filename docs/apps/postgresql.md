# PostgreSQL

## Diagnostic Mode

Bitnami's chart allow a diagnostic mode to be set that will disable all probes and let the container run a `sleep infinite` command so that it will not terminate.

To enable it, set this in the helm chart values:

```yaml
diagnosticMode.enabled: true
```

You can use this mode to do diagnostics/maintenance tasks on it w/o having the database run in the background.

## Run commands in diagnostic mode

If you need to run commands, eg. to have the database to run commands against, the default scripts live in `/opt/bitnami/scripts/postgresql/` and you need to use `entrypoint.sh` like so:

```shell
/opt/bitnami/scripts/postgresql/entrypoint.sh /opt/bitnami/scripts/postgresql/setup.sh

/opt/bitnami/scripts/postgresql/entrypoint.sh pg_upgrade -b /bitnami/postgresql/oldbin -B /opt/bitnami/postgresql/bin -d /bitnami/postgresql/olddata -D /bitnami/postgresql/data
```

## Upgrade

There's still not a way to handle this in a simple way with the helm chart, Bitnami just links to the official docs, that show 2 methods of doing it:
- A `pg_dumpall` followed by a `pg_restore` to backup the old instance, switch the chart to use the new version, and reimport the databases there.
- A `pg_upgrade` to do the upgrade in place.

### Proven unofficial way

[source](https://github.com/bitnami/charts/issues/8025#issuecomment-1708879837)

Duplicate volume and helm chart, set the new ones to the new version and start both

```shell
/opt/bitnami/scripts/postgresql/entrypoint.sh /bin/bash -c 'export PGPASSWORD="<OLD_POSTGRES_PASSWORD>"; echo "local all postgres trust" > /opt/bitnami/postgresql/conf/pg_hba.conf; pg_ctl reload; time pg_dumpall -h <OLD_POSTGRES_HOST> -U postgres | psql -U postgres'
```

Stop both, restore new volume as old one (backup and restore onto old name) upgrade old chart and delete the new volume and chart
