# What is Columnar?

## Columnar intro

Columnar storage is a key part of the data warehouse, but why is that? In this article we will review what columnar storage is and why it’s such a key part of the data warehouse.

### The heap table

By default, data in Postgres is stored in a heap table. Heap tables are arranged row-by-row, similar to how you would arrange data when creating a large spreadsheet. Data can be added forever by appending data to the end of the table.

In Postgres, heap tables are organized into _pages_. Each page holds 8kb of data. Each page holds pointers to the start of each row in the data.

![https://www.interdb.jp/pg/img/fig-1-04.png](https://www.interdb.jp/pg/img/fig-1-04.png)

_Image credit:_ [_The Internals of PostgreSQL_](https://www.interdb.jp/pg/pgsql01.html)

#### Advantages

Heap tables are optimized for transactional queries. Heap tables can use indexes to quickly find the row of data you are looking for — an index holds the page and row number for particular values of data. Generally, transactional workloads will read, insert, update, or delete small amounts of data at a time. Performance can scale so long as you have indexes to find the data you’re looking for.

#### Shortcomings

Heap tables perform poorly when data cannot be found by an index, known as a _table scan_. In order to find the data, all data in the table must be read. Because the data is organized by row, you must read each row to find it. When you active dataset size grows beyond the available memory on the system, you will find these queries slow down tremendously.

Additionally, scans assisted by an index can only go so far if you are requesting a large amount of data. For example, if you would like to know the average over a given month and have an index on the timestamp, the index will help Postgres find the relevant data, but it will still need to read every target row individually in the table to compute the average.

### Enter Columnar

Columnar tables are organized transversely from row tables. Instead of rows being added one after another, rows are inserted in bulk into a stripe. Within each stripe, data from each column is stored next to each other. Imagine rows of a table containing:

```yaml
| a | b | c | d |
| a | b | c | d |
| a | b | c | d |
```

This data would be stored as follows in columnar:

```yaml
| a | a | a |
| b | b | b |
| c | c | c |
| d | d | d |
```

In columnar, you can think of each stripe as a row of metadata that also holds up to 150,000 rows of data. Within each stripe, data is divided into _chunks_ of 1000 rows, and then within each chunk, there is a “row” for each _column_ of data you stored in it. Additionally, columnar stores the minimum, maximum, and count for each column in each chunk.

#### Advantages

Columnar is optimized for table scans — in fact, it doesn’t use indexes at all. Using columnar, it’s much quicker to obtain all data for a particular column. The database does not need to read data that you are not interested in. It also uses the metadata about the values in the chunks to eliminate reading data. This is a form of “auto-indexing” the data.

Because similar data is stored next to each other, very high data compression is possible. Data compression is an important benefit because columnar is often used to store huge amounts of data. By compressing the data, you can effectively read data more quickly from disk, which both reduces I/O and increasing effective fetch speed. It also has the effect of making better use of disk caching as data is cached in its compressed form. Lastly, you greatly reduce your storage costs.

#### Shortcomings

Columnar storage is not designed to do common transactional queries like “find by ID” - the database will need to scan a much larger amount of data to fetch this information than a row table.

Additionally, columnar storage on Hydra is an append-only datastore. While Hydra Columnar [supports updates and deletes](updates-and-deletes.md), it is not as performant as row tables.

Lastly, columnar tables need to be inserted in bulk in order to create efficient stripes. This makes them ideal for long-term storage of data you already have, but not ideal when data is still streaming into the database. For these reasons, it’s best to store data in row-based tables until it is ready to be archived into columnar.

## Using a columnar table

Create a Columnar table by specifying `USING columnar` when creating the table.

```sql
CREATE TABLE my_columnar_table
(
	id INT,
	i1 INT,
	i2 INT8,
	n NUMERIC,
	t TEXT
) USING columnar;
```

Insert data into the table and read from it like normal (subject to the limitations listed below). Note that columnar supports only btree and hash indexes and their associated constraints. Review columnar limitations in Hydra documentation.

## Converting From Row to Columnar

Hydra has a convenience function that will copy your row table to columnar.

```sql
CREATE TABLE my_table (i INT8);
-- convert to columnar

SELECT columnar.alter_table_set_access_method('my_table', 'columnar');
```

Data can also be converted manually by copying. For instance:

```sql
CREATE TABLE table_heap (i INT8);
CREATE TABLE table_columnar (LIKE table_heap) USING columnar;
INSERT INTO table_columnar SELECT * FROM table_heap;
```

## Partitioning

Columnar tables can be used as partitions; and a partitioned table may be made up of any combination of row and columnar partitions. You can use this feature to have archived data from previous months or years stored in columnar tables while active data is added to a heap table.

```sql
CREATE TABLE parent(ts timestamptz, i int, n numeric, s text)
PARTITION BY RANGE (ts);

-- columnar partition

CREATE TABLE p0 PARTITION OF parent
FOR VALUES FROM ('2020-01-01') TO ('2020-02-01')
USING COLUMNAR;

-- columnar partition

CREATE TABLE p1 PARTITION OF parent
FOR VALUES FROM ('2020-02-01') TO ('2020-03-01')
USING COLUMNAR;

-- row partition

CREATE TABLE p2 PARTITION OF parent
FOR VALUES FROM ('2020-03-01') TO ('2020-04-01');
INSERT INTO parent VALUES ('2020-01-15', 10, 100, 'one thousand'); -- columnar
INSERT INTO parent VALUES ('2020-02-15', 20, 200, 'two thousand'); -- columnar
INSERT INTO parent VALUES ('2020-03-15', 30, 300, 'three thousand'); -- row
```

When performing operations on a partitioned table with a mix of row and columnar partitions, take note of the following behaviors for operations that are supported on row tables but not columnar (e.g. tuple locks).

Note that the columnar engine supports `btree` and `hash` indexes (and the constraints requiring them) but does not support `gist`, `gin`, `spgist` and `brin` indexes. For this reason, if some partitions are columnar and if the index is not supported by columnar, then it's impossible to create indexes on the partitioned (parent) table directly. In that case, you need to create the index on the individual row partitions. Similarly for the constraints that require indexes, e.g.:

```sql
CREATE INDEX p2_ts_idx ON p2 (ts);
CREATE UNIQUE INDEX p2_i_unique ON p2 (i);
ALTER TABLE p2 ADD UNIQUE (n);
```
