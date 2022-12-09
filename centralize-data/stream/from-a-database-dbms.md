# From a database (DBMS)

### Populate data to Hydra with Foreign Data Wrappers (FDWs)

## Postgres External Tables

Postgres External Tables are implemented using `postgres_fdw`. To set up a Postgres External Table from `psql`, run the following SQL, replacing `...` with your server's information.

```sql
CREATE EXTENSION postgres_fdw;

CREATE SERVER remote_pg_server FOREIGN DATA WRAPPER postgres_fdw OPTIONS (host '...', port '...', dbname '...');

CREATE USER MAPPING FOR CURRENT_USER SERVER remote_pg_server OPTIONS (user '...', password '...');
```

You can now create foreign tables from your remote Postgres database. You can [import all of your remote tables](https://www.postgresql.org/docs/current/sql-importforeignschema.html) at once as follows:

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

## MySQL External Tables

MySQL External Tables are implemented using [`mysql_fdw`](https://github.com/EnterpriseDB/mysql\_fdw). To set up a MySQL External Table from `psql`, run the following SQL, replacing `...` with your server's information.

```sql
CREATE EXTENSION mysql_fdw;

CREATE SERVER remote_mysql_server FOREIGN DATA WRAPPER mysql_fdw OPTIONS (host '...', port '...');

CREATE USER MAPPING FOR CURRENT_USER SERVER remote_mysql_server OPTIONS (username '...', password '...');
```

You can now create foreign tables from your remote MySQL database. Assuming your remote MySQL database's name is `mysql`, you can [import all of your remote tables](https://www.postgresql.org/docs/current/sql-importforeignschema.html) at once as follows:

```sql
CREATE SCHEMA remote_mysql;

IMPORT FOREIGN SCHEMA mysql FROM SERVER remote_mysql_server INTO remote_mysql;
```

##

## Google Spreadsheet External Tables

You can run run queries against Google Spreadsheets. Google Spreadsheet External Tables are implemented using [`gspreadsheet_fdw`](https://github.com/HydrasDB/gspreadsheet\_fdw). To create a Google Spreadsheet External Table, create a Google Spreadsheet with some data:

* Put column names in the first row: untitled columns will not be read
* A blank row terminates the table (data below won't be read)
* Put it in the first (and only) worksheet

Get the spreadsheet ID from the HTTP URL. The ID is a 44-character string matching regexp `[A-Za-z0-9_]{44}`. It lives between the `/spreadsheets/d/` and possible trailing `/edit/blah` in the URL of your Google Spreadsheet.

Create a Google Service Account and enable Google Sheets API access by following [this guide](https://docs.gspread.org/en/latest/oauth2.html). Share your Google spreadsheet with the Google Service Account email that is in the format of `...@...gserviceaccount.com`.

Create a foreign table, replacing `...` with your Google Spreadsheet ID and Google Service Account credentials in JSON format:

```sql
CREATE EXTENSION multicorn;

CREATE SERVER multicorn_gspreadsheet FOREIGN DATA WRAPPER multicorn
  OPTIONS (
    wrapper 'gspreadsheet_fdw.GspreadsheetFdw'
  );

CREATE FOREIGN TABLE test_spreadsheet (
  id character varying,
  name   character varying
) server multicorn_gspreadsheet options(
  gskey '...',
  serviceaccount '...'
);
```

## Other External Table Engines

Hydra is working on providing connections to other data sources in the near future. If you have a request, please reach out to us via our support channel.

