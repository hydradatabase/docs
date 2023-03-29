# Using Hydra Columnar

## Should I use a row or columnar table?

Please see our documentation about [when to use row and columnar tables](../organize/data-modeling/row-vs-column-tables.md).

## Enabling Columnar

For Hydra cloud databases, columnar is already enabled.

For Hydra open source, the default `postgres` database has columnar enabled. Additional databases you create will need to have columnar enabled by running the query below as superuser:

```sql
CREATE EXTENSION IF NOT EXISTS columnar;
```

Once installed, you will not need superuser to utilize columnar tables.

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
