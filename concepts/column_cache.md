# Column Cache

The columnar store makes extensive use of compression, allowing
for your data to be stored efficiently on disk and in memory as
compressed data.

While this is very helpful, in some circumstances a long running
analytical query can retrieve and decompress the same columns on
a recurring basis.  In order to counter that, a column caching
mechanism is available to store uncompressed data in-memory, up
to a specified amount of memory per worker.

These uncompressed columns are then available for the lifetime
of a `SELECT` query, and are managed by the columnar store
directly.  NOTE that this cache is not used for `UPDATE`, `DELETE`,
or `INSERT`, and are released after the `SELECT` query is complete.

## Enabling the Cache

Caching is be enabled or disabled with a GUC: `columnar.enable_column_cache`.

```
set columnar.enable_column_cache = 't';
```

In addition, the cache size can be set, with a default of `200MB` per
process.

```
set columnar.column_cache_size = '2000MB';
```

This size can range between `20MB` and `20000MB`, and is consumed by
each parallel process.  This means that if you have `8` parallel processes
and set the cache size to `2000MB`, then a query can consume up to
`8 * 2000000` bytes, or up to `16GB` of RAM, so it is very important to
choose a cache size that is appropriate for your workload.
