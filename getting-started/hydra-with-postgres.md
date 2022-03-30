# 🐘 Hydra with Postgres

Hydra with Postgres is a managed Postgres database that is ready to scale to any size.

### Importing Data

You can use any Postgres tool to import data. The most common way is to use `pg_dump` and `pg_restore`. For example:

```shell
pg_dump -Fc --no-acl --no-owner \
    -h your.db.hostname \
    -U username \
    databasename > mydb.dump

pg_restore --verbose --clean --no-acl --no-owner \
    -h 623942e76454.db.hydras.io \
    -U uyuc0wnp2zmp \
    -d dida6lgi5dpl \
    mydb.dump
```

### Connecting and Running Queries

You can use any Postgres adapter or any tools with Postgres compatibility to connect and run queries.

### Disaster Recovery

All Hydra plans are monitored continuously by our infrastructure and will automatically recover in the case of hardware failure. All paid plans include HA allowing seamless instant recovery should there be a failure.

In addition, WAL (write-ahead logs) are backed up to an encrypted S3 bucket for 30 days. Should there be a need to restore your data from backup, we can restore your data to any point in the last 30 days. Should you need this service, please open a support ticket.

### Upgrading

To upgrade to a different plan size or to add Snowflake to an existing Hydra database, please contact support.
