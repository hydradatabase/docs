# Updates and Deletes

Hydra Columnar tables support updates and deletes, yet remains an append-only datastore. In order to achieve this,
Hydra maintains metadata about which rows in the table have been deleted or modified. Modified data is re-written
to the end of the table; you can think of an `UPDATE` as a DELETE followed by an `INSERT`.

When querying, Hydra will automatically return only the latest version of your data.

## Read Performance

To maximize performance of `SELECT` queries, columnar tables should have infrequent updates or deletes. Ideally, data that
is updated more frequently is best kept separate from archived data. This allows you to either store this data in row
tables, or rewrite the columnar table using `VACUUM FULL` which will re-optimize the data for reads.

Like an `INSERT`, each transaction with an `UPDATE` query will write a new stripe. To maximize the size of these stripes,
update as much data in a single transaction as possible.

## Write Performance

Each updates or deletes query will lock the table, meaning multiple `UPDATE` or `DELETE` queries on the same table will be
executed serially (one at a time). `UPDATE` queries rewrite any rows that are modified, and thus are relatively slow.
`DELETE` queries only modify metadata and thus complete quite quickly.

## Space Reclamation

Deleting data does not reclaim space, and updated data will also leave behind deleted data (the previous version of the
data). In order to reclaim space and re-compress your data, you must rewrite the table using `VACUUM FULL`. Note that
`VACUUM FULL` requires a full table lock and can take considerable time for larger tables.

Future versions of Hydra Columnar will address this shortcoming.

## Isolation

ℹ️ For terms used in this section, please refer to [the Postgres documentation on isolation levels][tx-iso].

[tx-iso]: https://www.postgresql.org/docs/current/transaction-iso.html

Hydra Columnar updates and deletes will meet the isolation level requested for your current transaction
(the default is `READ COMMITTED`).
Keep in mind that an `UPDATE` query is implenmented as an `DELETE` followed by an `INSERT`.
As new data that is inserted in one transaction can appear in a second transaction in `READ COMMITTED`,
this can affect concurrent transactions even if the first transaction was an `UPDATE`.
While this satisfies `READ COMMITTED`, it may result in unexpected behavior.
This is also possible in heap (row-based) tables,
but heap tables contain additional metadata that limit the impact of this case.

For stronger isolation guarantees, `REPEATABLE READ` is recommended,
as your transaction will be cancelled if it references data that has been modified by another transaction.
In this case, your application should be prepared to retry the transaction.
