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

Postgres External Tables are implemented using `postgres_fdw`. To set up a Postgres External Table from `psql`, run the following SQL, replacing `...` with your server's information.

```sql
CREATE EXTENSION postgres_fdw;

CREATE SERVER remote_pg_server FOREIGN DATA WRAPPER postgres_fdw OPTIONS (host '...', port '...', dbname '...');

CREATE USER MAPPING FOR CURRENT_USER SERVER remote_pg_server OPTIONS (user '...', password '...');
```

You can now create foreign tables from your remote Postgres database.
You can [import all of your remote tables](https://www.postgresql.org/docs/current/sql-importforeignschema.html) at once as follows:

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

## MySQL External Tables

MySQL External Tables are implemented using [`mysql_fdw`](https://github.com/EnterpriseDB/mysql_fdw).
To set up a MySQL External Table from `psql`, run the following SQL, replacing `...` with your server's information.

```sql
CREATE EXTENSION mysql_fdw;

CREATE SERVER remote_mysql_server FOREIGN DATA WRAPPER mysql_fdw OPTIONS (host '...', port '...');

CREATE USER MAPPING FOR CURRENT_USER SERVER remote_mysql_server OPTIONS (username '...', password '...');
```

You can now create foreign tables from your remote MySQL database.
Assuming your remote MySQL database's name is `mysql`, you can [import all of your remote tables](https://www.postgresql.org/docs/current/sql-importforeignschema.html) at once as follows:

```sql
CREATE SCHEMA remote_mysql;

IMPORT FOREIGN SCHEMA mysql FROM SERVER remote_mysql_server INTO remote_mysql;
```

## HTTP External Tables

You can call a HTTP web service to obtain data that lives outside Hydra and create a [view](https://www.postgresql.org/docs/current/sql-createview.html) to consume the data like a table.
HTTP external tables are implemented using [`pgsql-http`](https://github.com/pramsey/pgsql-http).

The following is an example that calls the GitHub organization repo endpoint to get a list of repos in JSON format and defines a view to hold the result:

```sql
CREATE EXTENSION http;

CREATE VIEW github_repos AS SELECT json_array_elements(content::json) as repo_json
  FROM http_get('https://api.github.com/orgs/hydrasdb/repos');
```

You can query the view as if it's a table. Note that Postgres view is read-only.

The following gets all the repo names from the view:


```sql
SELECT repo_json->'name' FROM github_repos;
```

There are other functions introduced by the `pgsql-http`. For example, run a GET request with an Authorization header:

```sql
SELECT content::json->'headers'->>'Authorization'
  FROM http((
          'GET',
           'http://httpbin.org/headers',
           ARRAY[http_header('Authorization','Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9')],
           NULL,
           NULL
        )::http_request)
```

Read the `status` and `content` fields out of a `http_response` object:

```sql
SELECT status, content_type
  FROM http_get('http://httpbin.org/');
```

Use the PUT command to send a simple JSON document to a server:

```sql
SELECT status, content_type, content::json->>'data' AS data
  FROM http_put('http://httpbin.org/patch', '{"this":"that"}', 'application/json');
```

As a shortcut to send data to a GET request, pass a JSONB data argument:

```sql
SELECT status, content::json->'args' AS args
  FROM http_get('http://httpbin.org/get',
                jsonb_build_object('myvar','myval','foo','bar'));
```

Here is a list of available functions:

* `http_header(field VARCHAR, value VARCHAR)` returns `http_header`
* `http(request http_request)` returns `http_response`
* `http_get(uri VARCHAR)` returns `http_response`
* `http_get(uri VARCHAR, data JSONB)` returns `http_response`
* `http_post(uri VARCHAR, content VARCHAR, content_type VARCHAR)` returns `http_response`
* `http_post(uri VARCHAR, data JSONB)` returns `http_response`
* `http_put(uri VARCHAR, content VARCHAR, content_type VARCHAR)` returns `http_response`
* `http_patch(uri VARCHAR, content VARCHAR, content_type VARCHAR)` returns `http_response`
* `http_delete(uri VARCHAR, content VARCHAR, content_type VARCHAR))` returns `http_response`
* `http_head(uri VARCHAR)` returns `http_response`
* `http_set_curlopt(curlopt VARCHAR, value varchar)` returns `boolean`
* `http_reset_curlopt()` returns `boolean`
* `http_list_curlopt()` returns `setof(curlopt text, value text)`
* `urlencode(string VARCHAR)` returns `text`
* `urlencode(data JSONB)` returns `text`
