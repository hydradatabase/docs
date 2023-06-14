# Vacuuming

The columnar store provides multiple methods to vacuum and clean up tables.
Among these are the standard `VACUUM` and `VACUUM FULL` functionality, but
also provided are UDF's, or User Defined Functions that help for incremental
vacuuming of large tables or tables with many holes.

## UDF's

Vacuuming requires an exclusive lock while the data that is part of the
table is reorganized and consolidated. This can cause other queries to
pause until the vacuum is complete, thus stalling other activity in the
database.

Using the vacuum UDF, you can specify the number of `stripes` to consolidate
in order to lower the amount of time where a table is locked.

```
SELECT columnar.vacuum('mytable', 25);
```

Using the optional stripe count argument, a vacuum can be performed
incrementally. A value will return of how many stripes were modified.
Calling this function repeatedly is fine, and it will continue to vacuum
the table until there is nothing more to do, and will return `0` as the
count.

| Parameter    | Description                 | Default        |
| ------------ | --------------------------- | -------------- |
| table        | Table name to vacuum        | none, required |
| stripe_count | Number of stripes to vacuum | `0`, or all    |

### Vacuuming All

In addition, you are provided a convenience function that can vacuum a
whole schema, pausing between each vacuum and table to unlock and allow
for other queries to continue on the database.

```
SELECT columnar.vacuum_full();
```

By default, this will vacuum the `public` schema, but there are other options
to control the vacuuming process.

| Parameter    | Description                                                                                   | Default  |
| ------------ | --------------------------------------------------------------------------------------------- | -------- |
| schema       | Which schema to vacuum, if you have more than one schema, multiple calls will need to be made | `public` |
| sleep_time   | The amount of time to sleep, in seconds, between vacuum calls                                 | `0.1`    |
| stripe_count | Number of stripes to vacuum per table between calls                                           | `25`     |

#### Examples

```
SELECT columnar.vacuum_full('public');
SELECT columnar.vacuum_full(sleep_time => 0.5);
SELECT columnar.vacuum_full(stripe_count => 1000);
```
