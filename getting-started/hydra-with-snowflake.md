# ❄ Hydra with Snowflake

## Importing Data

### Import via Airbyte

You can import data directly into your Snowflake instance using Airbyte. For instructions, see our [Airbyte guide](../tools/airbyte.md).

### Import via Hydra with Postgres

You may find it easier to first import data into Hydra with Postgres, then move your data to Snowflake using Hydra. Hydra staff will help you move you data from Postgres to Snowflake. We currently partner with Airbyte to move data between data warehouse engines.

Please contact Hydra if you need a custom Hydra with Postgres plan with additional resources during your migration.

### Direct Import

This process is for importing your data from an existing Postgres database directly into a new Hydra with Snowflake data warehouse. We recommend reading up on [some best practices](https://hevodata.com/blog/snowflake-etl-best-practices-cloud-data-warehouse/) when moving data into Snowflake for the first time.

{% hint style="warning" %}
This process is complex and will be improved with additional tooling soon. Hydra staff are happy to provide hands-on help in the meantime.
{% endhint %}

#### **1. Import your Postgres schema**

The first step is to import your existing schema into Hydra directly. This is a cached copy of your schema that is kept in Postgres.

Get a plain text dump of your existing `public` schema using:

```shell
pg_dump -Fp \
    --no-acl --no-owner --schema-only --no-comments \
    --quote-all-identifiers \
    -n public \
    -h your.db.hostname \
    -U username \
    databasename > myschema.sql
```

You may wish to review this file to assure it only contains what you wish to import into Snowflake.

Restore this schema dump into Hydra using the following command. The `SET...` command will disable the Snowflake connection so you can manipulate the cached .

```
echo "SET hydra.snowflake_enabled = false;" |
  cat - myschema.sql |
  psql "postgres://uXXX:YYY@ZZZ.db.hydras.io/dAAA"
```

#### **2. Import your Snowflake schema**

For Snowflake, your SQL file should only be the tables in your schema. Remove any other items from the dump like `SET` statements, etc.

Depending on the data types in your schema, you may need to make some changes to import it into Snowflake. We recommend reviewing [this table of data types](https://hevodata.com/blog/postgresql-to-snowflake-etl/#data-type-conversion) and making manual changes to your schema. The data types must be compatible with the data types you've specified in Postgres. Generally, it is safe to map any complex Postgres types to `text` these will be cast to their appropriate type by Postgres when the data is returned.

{% hint style="warning" %}
While the queries will execute on Snowflake, all queries are first parsed by Postgres and must be valid Postgres queries. Use data type names that are valid on both platforms, like `text`, rather than `string`. If Snowflake-specific syntax or data types are needed, see \[Executing Snowflake-specific Queries].
{% endhint %}

Once edited, you can import your schema directly with Hydra.

```
psql "postgres://uXXX:YYY@ZZZ.db.hydras.io/dAAA" < myschema.sql
```

#### **3. Copy data**

Generally, there are two ways to move the data: SQL import or CSV import.

For larger data sets, a CSV import is recommended. Follow the [standard method documented by Snowflake](https://community.snowflake.com/s/article/PostgreSQL-to-Snowflake-ETL-Steps-to-Migrate-Data). Data is uploaded to S3, staged, and then imported.

A direct SQL import is possible with Hydra for smaller data sets, however Snowflake's slow `INSERT` performance will make this run very slowly. You can split tables into separate files to gain concurrency (use pg\_dump's `-t table`).

```
pg_dump --data-only \
  -n public \
  --rows-per-insert=100 \
  --quote-all-identifiers \
  -h your.db.hostname \
  -U username \
  databasename > data.sql
```

This can then be imported into Hydra:

```
psql "postgres://uXXX:YYY@ZZZ.db.hydras.io/dAAA" < data.sql
```

## Connecting and Running Queries

You can use any Postgres adapter or any tools with Postgres compatibility to connect and run queries. Queries are not rewritten and must be compatible with both Postgres and Snowflake compatible. If you need to use a Snowflake-specific, see below.

#### Executing Snowflake-specific Queries

Generally, we recommend using agnostic SQL whenever possible - most queries run fine in both Postgres and Snowflake. However, you may need to use specific features or functions only available in Snowflake. In order to execute Snowflake SQL, you must wrap the SQL in a function. This function has the signature:

```sql
hydra.run_on_snowflake(db_name text, schema_name text, query text)
RETURNS SETOF record
```

Please ask Hydra staff for your Snowflake `db_name`- this is not the same as your Postgres database used in your credential. Schema name is typically `public` unless you have used multiple schemas.

While you can call the function directly in a `SELECT` clause, the recommended way to treat the function like a table as follows:

```sql
SET hydra.snowflake_enabled = false;

SELECT t1.*
FROM
  hydra.run_on_snowflake('EXAMPLE', 'PUBLIC', 'SELECT 5, 10, 15, 20')
  AS t1(i1 numeric, i2 numeric, i3 numeric, i4 numeric);
  
SET hydra.snowflake_enabled = true;
```

{% hint style="warning" %}
You must temporarily disable Snowflake as the function must be run on Postgres.
{% endhint %}
