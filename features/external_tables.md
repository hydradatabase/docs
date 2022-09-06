---
description: >-
  External tables allow you to read and write data between Hydra and other data
  sources.
---

# 📡 External Tables

External Tables are data that lives in databases outside Hydra. You can use External Tables to select, join, and insert data to and from your data warehouse. For example, in the image below, we see Hydra tables represented as “local” and RDS tables as “external.” Data in RDS is being fetched and returned through Hydra.

<figure><img src="../.gitbook/assets/fdw_diagram.jpg" alt=""><figcaption><p>External Tables can be used to query and combine data from multiple data sources.</p></figcaption></figure>

## Use Cases

Data warehouses are referred to as an organization’s source of truth because they aggregate data from different sources into a single, central data store to support data analysis. Traditionally, organizations configure data pipelines to copy data from multiple sources into the warehouse. While this approach works well for immutable data like analytics, rapidly changing data quickly becomes out of date. Constantly refreshing this data is expensive, slow, and restrictive.

External tables address the following problems:

* Data copying grows storage and pipeline costs.
* The warehouse’s data recency is limited to the data copy rate.
* Impossible to run ad-hoc queries without copying relevant tables to the warehouse.

With External Tables, queries execute on source databases directly, which greatly reduces frictions and costs outlined above. In our first example, we’ll show how to query RDS tables from Hydra.

## Postgres External Tables

Postgres External Tables are implemented using `postgres_fdw`. To set up a Postgres External Table from `psql`, run the following SQL, replacing `...` with you server's information.

```sql
CREATE EXTENSION postgres_fdw;

CREATE SERVER remote_pg_server FOREIGN DATA WRAPPER postgres_fdw OPTIONS (host '...', port '...', dbname '...');

CREATE USER MAPPING FOR CURRENT_USER SERVER rds_server OPTIONS (user '...', password '...');
```

You can now create foreign tables from your remote database. You can [import all of your remote tables](https://www.postgresql.org/docs/current/sql-importforeignschema.html) at once as follows:

```sql
CREATE SCHEMA remote_pg;

IMPORT FOREIGN SCHEMA public FROM SERVER remote_pg_server INTO remote_pg;
```

💡 You can name the server and schema as you see fit.

You can now query your remote database from Hydra, including:

* Using `JOIN` between data in Hydra and in your remote database
* Inserting, updating, or deleting data

### Bi-directional connection

On many Postgres providers, you can also set up this connection in the other direction. This allows you to access your Hydra data warehouse from your other databases. You may need to contact your provider to get the necessary permissions.

## Other External Table Engines

Hydra is working on providing connections to other data sources in the near future. If you have a request, please reach out to us via our support channel.
