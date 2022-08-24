---
description: Cron is used to schedule tasks at a regular interval on your data warehouse.
---

# 🔁 Cron

Cron (or `pg_cron`) is a cron-style task scheduler that can run arbitrary SQL inside your data warehouse on any schedule. You can use cron to move data between data sources (using foreign data wrappers), move data from row to [columnar tables](/features/columnar.md), perform transformations, refresh materialized views, or any other regular maintenance tasks.

Cron uses the standard cron syntax:

```
 ┌───────────── min (0 - 59)
 │ ┌────────────── hour (0 - 23)
 │ │ ┌─────────────── day of month (1 - 31)
 │ │ │ ┌──────────────── month (1 - 12)
 │ │ │ │ ┌───────────────── day of week (0 - 6) (0 to 6 are Sunday to
 │ │ │ │ │                  Saturday, or use names; 7 is also Sunday)
 │ │ │ │ │
 │ │ │ │ │
 * * * * *
```

In addition, you can use `@hourly`, `@daily`, `@weekly`, `@monthly`, and `@yearly` shortcuts. These are equivalent to:

```
0 * * * *    @hourly
0 0 * * *    @daily
0 0 * * 0    @weekly
0 0 1 * *    @monthly
0 0 1 1 *    @yearly
```

Note that the timezone used for all tasks use UTC. 

## Administering tasks

All administration of tasks occur in the `postgres` database. Your Hydra user has permission to connect to this database for this purpose. 

To connect to the database, connect normally, then switch to `postgres` using `\c postgres`.

```console
$ psql service=hydra

d123456=> \c postgres

psql (13.7, server 13.6 (Ubuntu 13.6-1.pgdg18.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "postgres" as user "u123456".
postgres=>
```

### Creating a task

To create a task:

```sql
SELECT cron.schedule_in_database(
  'refresh-abc', -- task name,
  '30 * * * *', -- schedule
  'd123456', -- database
  $$ REFRESH MATERIALIZED VIEW CONCURRENTLY abc $$ -- command
);
```

### Deleting a task

To delete a task:

```sql
SELECT cron.unschedule('refresh-abc');
```

### Review tasks

The `cron.job` table contains all current jobs and their parameters. You can also use this table to obtain the underlying `jobid`.

```sql
SELECT * FROM cron.job;
```

#### Reviewing task logs

Each time a job runs, a record is placed in `cron.job_run_details`.

```sql
SELECT * FROM cron.job_run_details
ORDER BY start_time DESC
LIMIT 10
```

#### Deleting task logs

Cleaning the task logs is at your discretion. You could schedule a task to remove logs after a set time:

```sql
SELECT cron.schedule_in_database(
  'clean-cron-logs',
  '0 0 * * *',
  'postgres',
  $$ DELETE FROM cron.job_run_details WHERE end_time < now() - interval '7 days' $$
);
```

### Modify, enable, or disable a task

You can alter any parameters about a task, such as its schedule, command, and whether it is currently active.

First you must obtain the job ID:

```sql
SELECT cron.jobid FROM cron.job WHERE jobname = 'refresh-abc';
```

You can then alter the task. `NULL` indicates that change is desired. The last parameter controls whether the task is disabled:

```sql
SELECT cron.alter_job(
  42,   -- job id
  NULL, -- schedule, NULL = do not change
  NULL, -- command, NULL = do not change
  NULL, -- database, NULL = do not change
  NULL, -- username, NULL = do not change
  false -- active, false=inactive, true=active
)
```