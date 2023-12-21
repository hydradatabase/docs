---
description: >-
  Hydra supports traditional materialized views using both row and columnar tables,
  as well as incremental materialized views powered by pg_ivm.
---

# Materialized Views

Unlike regular views, which are virtual tables that run a query each time they are accessed,
[materialized views](https://www.postgresql.org/docs/15/sql-creatematerializedview.html) are precomputed database tables
that store the results of a query and only update them when they are directly refreshed. Compared to performing
equivalent queries on the underlying tables, materialized views can significantly enhance the speed of your queries,
particularly for more complex queries.

Materialized views are typically used in scenarios where query performance is critical, queries are executed frequently
against large and complex data sets, and the data being queried doesn't change as frequently. In cases where your
queried data requires multiple aggregations, joins, and operations that are computed frequently for reporting or
analytics, you can efficiently improve performance and reduce execution time using precomputed materialized views.

You can create a materialized view using the following command:

```sql
CREATE MATERIALIZED VIEW [ IF NOT EXISTS ] view_name
    [ (column_name [, ...] ) ]
    [ USING method ]
    AS query
    [ WITH [ NO ] DATA ]
```

* You can set `USING columnar` to store large materialized views in columnar for fast analytics
* You can set `USING heap` for traditional row-based materialized views

For example, here's a materialized view that computes a monthly sales summary containing the total products sold and
sales gained:

```sql
CREATE MATERIALIZED VIEW sales_summary
  USING columnar
AS
SELECT
    date_trunc('month', order_date) AS month,
    product_id,
    SUM(quantity) AS total_quantity,
    SUM(quantity * price) AS total_sales
FROM
    sales
GROUP BY
    date_trunc('month', order_date),
    product_id;
```

You can refresh your materialized view with the latest data using `REFRESH MATERIALIZED VIEW [ CONCURRENTLY ]
view_name`. Using the `CONCURRENTLY` option will allow the view to be refreshed while allowing queries to continue on
the current data, but requires a unique index on the view.

### Best Practices for Materialized Views

There are a couple of things you can do to make sure you're making the most of materialized views:

* **Refresh your materialized view:** The frequency and methods you employ to refresh your materialized view are crucial
  factors that influence the overall effectiveness of your view. As refreshing a materialized view can be a
  resource-intensive operation, it's better suited to data that updates infrequently.
* **Monitor disk space and performance:** Materialized views can consume significant amounts of disk space, so it's
  essential to monitor their size and delete unused ones to free up space. You should also monitor their refresh
  performance to ensure that they don't negatively impact the overall performance.
* **Add indexes to optimize your queries:** Materialized views offer a distinct advantage by being stored as
  regular tables in Postgres. This means that they can fully leverage the benefits of indexing
  techniques, leading to enhanced performance and efficient processing of large data sets.


## Incremental Materialized Views with pg_ivm

_Documentation in this section is courtesy [`pg_ivm`](https://github.com/sraoss/pg_ivm)._

`pg_ivm` provides a way to make materialized views up-to-date where only incremental changes are computed and applied
on views rather than recomputing the contents from scratch as `REFRESH MATERIALIZED VIEW` does. This process is
called "incremental view maintenance" (or IVM), and it can update materialized views more efficiently than
recomputation when only small parts of the view are changed.

There are two approaches with regard to timing of view maintenance: immediate and deferred. In immediate maintenance,
views are updated in the same transaction that its base table is modified.  In deferred maintenance, views are updated
after the transaction is committed, for example, when the view is accessed, as a response to user command like
`REFRESH MATERIALIZED VIEW`, or periodically in background, and so on. `pg_ivm` provides a kind of immediate
maintenance, in which materialized views are updated immediately in AFTER triggers when a base table is modified.

To begin using `pg_ivm`, you must first enable it on your database:

```sql
CREATE EXTENSION IF NOT EXISTS pg_ivm;
```

We call a materialized view supporting IVM an Incrementally Maintainable Materialized View (IMMV). To create an IMMV,
you have to call `create_immv` function with a relation name and a view definition query. For example:

```sql
SELECT create_immv('sales_test', 'SELECT * FROM sales');
```

creates an IMMV with name 'sales_test' defined as `SELECT * FROM sales`. This corresponds to the following command
to create a normal materialized view:

```sql
CREATE MATERIALIZED VIEW sales_test AS SELECT * FROM sales;
```

When an IMMV is created, some triggers are automatically created so that the view's contents are immediately updated
when its base tables are modified.

```sql
postgres=# SELECT create_immv('m', 'SELECT * FROM t0');
NOTICE:  could not create an index on immv "m" automatically
DETAIL:  This target list does not have all the primary key columns, or this view does not contain DISTINCT clause.
HINT:  Create an index on the immv for efficient incremental maintenance.
 create_immv
-------------
           3
(1 row)

postgres=# SELECT * FROM m;
 i
---
 1
 2
 3
(3 rows)

postgres=# INSERT INTO t0 VALUES (4);
INSERT 0 1
postgres=# SELECT * FROM m; -- automatically updated
 i
---
 1
 2
 3
 4
(4 rows)
```

### IMMV and Columnar

You can create an IMMV using columnar store by setting the default table access method. If the source table is columnar, you must also disable parallelism temporarily to create the IMMV:

```sql
SET default_table_access_method = 'columnar';
SET max_parallel_workers = 1;
SELECT create_immv('sales_test', 'SELECT * FROM sales');
```

After the IMMV is created, you can restore the settings to their defaults. (These settings do not affect other connections and will also be restored to their defaults upon your next session.)

```sql
SET max_parallel_workers = DEFAULT;
SET default_table_access_method = DEFAULT;
```

### Functions

#### create_immv

Use `create_immv` function to create IMMV.
```
create_immv(immv_name text, view_definition text) RETURNS bigint
```
`create_immv` defines a new IMMV of a query. A table of the name `immv_name` is created and a query specified by
`view_definition` is executed and used to populate the IMMV. The query is stored in `pg_ivm_immv`, so that it can be
refreshed later upon incremental view maintenance. `create_immv` returns the number of rows in the created IMMV.

When an IMMV is created, triggers are automatically created so that the view's contents are immediately updated when
its base tables are modified. In addition, a unique index is created on the IMMV automatically if possible.  If the
view definition query has a GROUP BY clause, a unique index is created on the columns of GROUP BY expressions. Also,
if the view has DISTINCT clause, a unique index is created on all columns in the target list. Otherwise, if the IMMV
contains all primary key attributes of its base tables in the target list, a unique index is created on these
attributes. In other cases, no index is created.

#### refresh_immv

Use `refresh_immv` function to refresh IMMV.
```
refresh_immv(immv_name text, with_data bool) RETURNS bigint
```

`refresh_immv` completely replaces the contents of an IMMV as `REFRESH MATERIALIZED VIEW` command does for a
materialized view. To execute this function you must be the owner of the IMMV.  The old contents are discarded.

The with_data flag is corresponding to `WITH [NO] DATA` option of `REFRESH MATERIALIZED VIEW` command. If with_data is
true, the backing query is executed to provide the new data, and if the IMMV is unpopulated, triggers for maintaining
the view are created. Also, a unique index is created for IMMV if it is possible and the view doesn't have that yet.
If with_data is false, no new data is generated and the IMMV become unpopulated, and the triggers are dropped from
the IMMV. Note that unpopulated IMMV is still scannable although the result is empty. This behaviour may be changed
in future to raise an error when an unpopulated IMMV is scanned.

#### get_immv_def

`get_immv_def` reconstructs the underlying SELECT command for an IMMV. (This is a decompiled reconstruction, not the
original text of the command.)

```
get_immv_def(immv regclass) RETURNS text
```

#### IMMV metadata catalog

The catalog `pg_ivm_immv` stores IMMV information.

| Name        | Type     | Description                                                                         |
| :---------- | :------- | :---------------------------------------------------------------------------------- |
| immvrelid   | regclass | The OID of the IMMV                                                                 |
| viewdef     | text     | Query tree (in the form of a nodeToString() representation) for the view definition |
| ispopulated | bool     | True if IMMV is currently populated                                                 |


### Example

In general, IMMVs allow faster updates than `REFRESH MATERIALIZED VIEW` at the price of slower updates to their base
tables. Update of base tables is slower because triggers will be invoked and the IMMV is updated in triggers per
modification statement.

For example, suppose a normal materialized view defined as below:

```sql
test=# CREATE MATERIALIZED VIEW mv_normal AS
        SELECT a.aid, b.bid, a.abalance, b.bbalance
        FROM pgbench_accounts a JOIN pgbench_branches b USING(bid);
SELECT 10000000
```

Updating a tuple in a base table of this materialized view is rapid but the `REFRESH MATERIALIZED VIEW` command on this
view takes a long time:

```sql
test=# UPDATE pgbench_accounts SET abalance = 1000 WHERE aid = 1;
UPDATE 1
Time: 9.052 ms

test=# REFRESH MATERIALIZED VIEW mv_normal ;
REFRESH MATERIALIZED VIEW
Time: 20575.721 ms (00:20.576)
```

On the other hand, after creating IMMV with the same view definition as below:

```sql
test=# SELECT create_immv('immv',
        'SELECT a.aid, b.bid, a.abalance, b.bbalance
         FROM pgbench_accounts a JOIN pgbench_branches b USING(bid)');
NOTICE:  created index "immv_index" on immv "immv"
 create_immv
-------------
    10000000
(1 row)
```

updating a tuple in a base table takes more than the normal view, but its content is updated automatically and this
is faster than the `REFRESH MATERIALIZED VIEW` command.

```sql
test=# UPDATE pgbench_accounts SET abalance = 1234 WHERE aid = 1;
UPDATE 1
Time: 15.448 ms

test=# SELECT * FROM immv WHERE aid = 1;
 aid | bid | abalance | bbalance
-----+-----+----------+----------
   1 |   1 |     1234 |        0
(1 row)
```

An appropriate index on IMMV is necessary for efficient IVM because we need to looks for tuples to be updated in IMMV.
If there are no indexes, it will take a long time.

Therefore, when an IMMV is created by the `create_immv` function, a unique index is created on it automatically if
possible. If the view definition query has a GROUP BY clause, a unique index is created on the columns of GROUP BY
expressions. Also, if the view has DISTINCT clause, a unique index is created on all columns in the target list.
Otherwise, if the IMMV contains all primary key attributes of its base tables in the target list, a unique index is
created on these attributes.  In other cases, no index is created.

In the previous example, a unique index "immv_index" is created on aid and bid columns of "immv", and this enables the
rapid update of the view. Dropping this index make updating the view take a loger time.

```sql
test=# DROP INDEX immv_index;
DROP INDEX

test=# UPDATE pgbench_accounts SET abalance = 9876 WHERE aid = 1;
UPDATE 1
Time: 3224.741 ms (00:03.225)
```

### Supported View Definitions and Restriction

Currently, IMMV's view definition can contain inner joins, DISTINCT clause, some built-in aggregate functions, simple
sub-queries in `FROM` clause, and simple CTE (`WITH` query). Inner joins including self-join are supported, but outer
joins are not supported. Supported aggregate functions are count, sum, avg, min and max. Other aggregates, sub-queries
which contain an aggregate or `DISTINCT` clause, sub-queries in other than `FROM` clause, window functions, `HAVING`,
`ORDER BY`, `LIMIT`/`OFFSET`, `UNION`/`INTERSECT`/`EXCEPT`, `DISTINCT ON`, `TABLESAMPLE`, `VALUES`, and
`FOR UPDATE`/`SHARE` can not be used in view definition.

The base tables must be simple tables. Views, materialized views, inheritance parent tables, partitioned tables,
partitions, and foreign tables can not be used.

The targetlist cannot contain system columns, columns whose name starts with `__ivm_`.

Logical replication is not supported, that is, even when a base table at a publisher node is modified, IMMVs at
subscriber nodes defined on these base tables are not updated.

### Limitiations and Restrictions

#### Aggregates

Supported aggregate functions are `count`, `sum`, `avg`, `min`, and `max`. Currently, only built-in aggregate functions
are supported and user defined aggregates cannot be used.

When an IMMV including aggregate is created, some extra columns whose name start with `__ivm` are automatically added
to the target list. `__ivm_count__` contains the number of tuples aggregated in each group. In addition, more than one
extra columns for each column of aggregated value  are added in order to maintain the value. For example, columns
named like  `__ivm_count_avg__` and `__ivm_sum_avg__` are added for maintaining an average value. When a base table is
modified, the new aggregated values are incrementally calculated using the old aggregated values and values of related
extra columns stored in the IMMV.

Note that for `min` or `max`, the new values could be re-calculated from base tables with regard to the affected groups
when a tuple containing the current minimal or maximal values are deleted from a base table. Therefore, it can takes a
long time to update an IMMV containing these functions.

Also, note that using `sum` or `avg` on `real` (`float4`) type or `double precision` (`float8`) type in IMMV is unsafe,
because aggregated values in IMMV can become different from results calculated from base tables due to the limited
precision of these types. To avoid this problem, use the `numeric` type instead.

#### Restrictions on Aggregate

If we have a `GROUP BY` clause, expressions specified in `GROUP BY` must appear in the target list.  This is how tuples
to be updated in the IMMV are identified. These attributes are used as scan keys for searching tuples in the IMMV,
so indexes on them are required for efficient IVM.

Targetlist cannot contain expressions which contain an aggregate in it.

#### Subqueries

Simple subqueries in `FROM` clause are supported.

#### Restrictions on Subqueries

Subqueries can be used only in `FROM` clause. Subqueries in target list or `WHERE` clause are not supported.

Subqueries containing an aggregate function or `DISTINCT` are not supported.

### CTE

Simple CTEs (`WITH` queries) are supported.

#### Restrictions on CTEs

`WITH` queries containing an aggregate function or `DISTINCT` are not supported.

Recursive queries (`WITH RECURSIVE`) are not allowed. Unreferenced CTEs are not allowed either, that is, a CTE must be
referenced at least once in the view definition query.

#### DISTINCT

`DISTINCT` is allowed in IMMV's definition queries. Suppose an IMMV defined with DISTINCT on a base table containing
duplicate tuples.  When tuples are deleted from the base table, a tuple in the view is deleted if and only if the
multiplicity of the tuple becomes zero.  Moreover, when tuples are inserted into the base table, a tuple is inserted
into the view only if the same tuple doesn't already exist in it.

Physically, an IMMV defined with `DISTINCT` contains tuples after eliminating duplicates, and the multiplicity of each
tuple is stored in a extra column named `__ivm_count__` that is added when such IMMV is created.

#### TRUNCATE

When a base table is truncated, the IMMV is also truncated and the contents become empty if the view definition query
does not contain an aggregate without a `GROUP BY` clause. Aggregate views without a `GROUP BY` clause always have one
row. Therefore, in such cases, if a base table is truncated, the IMMV is simply refreshed instead of being truncated.

#### Concurrent Transactions

Suppose an IMMV is defined on two base tables and each table was modified in different a concurrent transaction
simultaneously. In the transaction which was committed first, the IMMV can be updated considering only the change which
happened in this transaction. On the other hand, in order to update the IMMV correctly in the transaction which was
committed later, we need to know the changes occurred in both transactions.  For this reason, `ExclusiveLock` is held on
an IMMV immediately after a base table is modified in `READ COMMITTED` mode to make sure that the IMMV is updated in the
latter transaction after the former transaction is committed.  In `REPEATABLE READ` or `SERIALIZABLE` mode, an error is
raised immediately if lock acquisition fails because any changes which occurred in other transactions are not be visible
in these modes and IMMV cannot be updated correctly in such situations. However, as an exception if the IMMV has only
one base table and doesn't use DISTINCT or GROUP BY, and the table is modified by `INSERT`, then the lock held on the
IMMV is `RowExclusiveLock`.

#### Row Level Security

If some base tables have row level security policy, rows that are not visible to the materialized view's owner are
excluded from the result.  In addition, such rows are excluded as well when views are incrementally maintained.
However, if a new policy is defined or policies are changed after the materialized view was created, the new policy will
not be applied to the view contents.  To apply the new policy, you need to recreate IMMV.

#### How to Disable or Enable Immediate Maintenance

IVM is effective when we want to keep an IMMV up-to-date and small fraction of a base table is modified infrequently.
Due to the overhead of immediate maintenance, IVM is not effective when a base table is modified frequently.  Also, when
a large part of a base table is modified or large data is inserted into a base table, IVM is not effective and the cost
of maintenance can be larger than refresh from scratch.

In such situation, we can use `refesh_immv` function with `with_data = false` to disable immediate maintenance before
modifying a base table. After a base table modification, call `refresh_immv`with `with_data = true` to refresh the view
data and enable immediate maintenance.