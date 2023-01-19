---
description: Hydra integration
images:
- serialized/26-1.png
jupyblog:
  execute_code: false
jupyter:
  jupytext:
    formats: ipynb,md
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.14.4
  kernelspec:
    display_name: Python 3 (ipykernel)
    language: python
    name: python3
title: Hydra
---

## Hydra

In this tutorial, we'll show you how to use [Hydra](https://hydra.so/) (an open-source, Postgres-compatible data warehouse) and [JupySQL](https://github.com/ploomber/jupysql) to analyze large datasets efficiently.

<!-- #region -->
## Requirements


To run this tutorial, you need to install the following Python packages:
<!-- #endregion -->

```python
%pip install jupysql pandas pyarrow psycopg2-binary --quiet
```

*Note:* to run the `\copy` command, you need `pgspecial<2`, we recommend you getting it via `conda install`; since `pip install` might give you some issues.

```python
%conda install "pgspecial<2" -c conda-forge -y --quiet
```

You also need Docker installed and running to start the Hydra instance.


## Starting a Hydra instance

Let's fetch the docker image and start the Hydra instance. The next command will take a couple of minutes as it needs to fetch the image from the repository:

```sh
docker run --name hydra -e POSTGRES_DB=db \
  -e POSTGRES_USER=user \
  -e POSTGRES_PASSWORD=password \
  -p 5432:5432 -d ghcr.io/hydrasdb/hydra
```

<!-- #region -->


**Console output (1/1):**

```txt
fd21cec1f520e7a992c09c038a74f01a3611e3ab75d08dc4011447eba9a654ca
```

<!-- #endregion -->

## Data download

Now, let's fetch some sample data. We'll be using the [NYC taxi dataset](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page):

```python
import pandas as pd

df = pd.read_parquet("https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2021-01.parquet")
print(f"Number of rows: {len(df):,}")

# we convert it to CSV so we can upload it using the \COPY postgres command
df.to_csv("taxi.csv", index=False)
```

<!-- #region -->


**Console output (1/1):**

```txt
Number of rows: 1,369,769
```

<!-- #endregion -->

As you can see, this dataset contains ~1.4M rows.


## Uploading data to Hydra

We're ready to upload our data; we'll load the [JupySQL](https://github.com/ploomber/jupysql) extension and start the database connection:

```python
%load_ext sql
```

```python
%sql postgresql://user:password@localhost/db
```

Let's create the table; note that Hydra adds a [`USING columnar`](https://docs.hydras.io/concepts/what-is-columnar) option to the `CREATE TABLE` statement, which will optimize storage for analytical queries.

```sql
CREATE TABLE "taxi" (
    "VendorID" DECIMAL NOT NULL,
    tpep_pickup_datetime TIMESTAMP WITHOUT TIME ZONE,
    tpep_dropoff_datetime TIMESTAMP WITHOUT TIME ZONE,
    passenger_count DECIMAL,
    trip_distance DECIMAL NOT NULL,
    "RatecodeID" DECIMAL,
    store_and_fwd_flag BOOLEAN,
    "PULocationID" DECIMAL NOT NULL,
    "DOLocationID" DECIMAL NOT NULL,
    payment_type DECIMAL NOT NULL,
    fare_amount DECIMAL NOT NULL,
    extra DECIMAL NOT NULL,
    mta_tax DECIMAL NOT NULL,
    tip_amount DECIMAL NOT NULL,
    tolls_amount DECIMAL NOT NULL,
improvement_surcharge DECIMAL NOT NULL,
    total_amount DECIMAL NOT NULL,
    congestion_surcharge DECIMAL,
    airport_fee DECIMAL
) USING columnar;
```

<!-- #region -->


**Console output (1/2):**

```txt
*  postgresql://user:***@localhost/db
Done.
```

**Console output (2/2):**

```txt
[]
```

<!-- #endregion -->

Let's now upload the data:

```sql
\copy taxi from 'taxi.csv' WITH DELIMITER ',' CSV HEADER;
```

<!-- #region -->


**Console output (1/2):**

```txt
*  postgresql://user:***@localhost/db
0 rows affected.
```

**Console output (2/2):**

<table>
    <tr>
    </tr>
</table>
<!-- #endregion -->

Let's now query our data:

```sql
SELECT COUNT(*) FROM taxi
```

<!-- #region -->


**Console output (1/2):**

```txt
*  postgresql://user:***@localhost/db
1 rows affected.
```

**Console output (2/2):**

<table>
    <tr>
        <th>count</th>
    </tr>
    <tr>
        <td>1369769</td>
    </tr>
</table>
<!-- #endregion -->

We see that the ~1.4M are there. Let's take a look at the first rows:

```sql
SELECT * FROM taxi
LIMIT 3
```

<!-- #region -->


**Console output (1/2):**

```txt
*  postgresql://user:***@localhost/db
3 rows affected.
```

**Console output (2/2):**

<table>
    <tr>
        <th>VendorID</th>
        <th>tpep_pickup_datetime</th>
        <th>tpep_dropoff_datetime</th>
        <th>passenger_count</th>
        <th>trip_distance</th>
        <th>RatecodeID</th>
        <th>store_and_fwd_flag</th>
        <th>PULocationID</th>
        <th>DOLocationID</th>
        <th>payment_type</th>
        <th>fare_amount</th>
        <th>extra</th>
        <th>mta_tax</th>
        <th>tip_amount</th>
        <th>tolls_amount</th>
        <th>improvement_surcharge</th>
        <th>total_amount</th>
        <th>congestion_surcharge</th>
        <th>airport_fee</th>
    </tr>
    <tr>
        <td>1</td>
        <td>2021-01-01 00:30:10</td>
        <td>2021-01-01 00:36:12</td>
        <td>1.0</td>
        <td>2.1</td>
        <td>1.0</td>
        <td>False</td>
        <td>142</td>
        <td>43</td>
        <td>2</td>
        <td>8.0</td>
        <td>3.0</td>
        <td>0.5</td>
        <td>0.0</td>
        <td>0.0</td>
        <td>0.3</td>
        <td>11.8</td>
        <td>2.5</td>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
        <td>2021-01-01 00:51:20</td>
        <td>2021-01-01 00:52:19</td>
        <td>1.0</td>
        <td>0.2</td>
        <td>1.0</td>
        <td>False</td>
        <td>238</td>
        <td>151</td>
        <td>2</td>
        <td>3.0</td>
        <td>0.5</td>
        <td>0.5</td>
        <td>0.0</td>
        <td>0.0</td>
        <td>0.3</td>
        <td>4.3</td>
        <td>0.0</td>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
        <td>2021-01-01 00:43:30</td>
        <td>2021-01-01 01:11:06</td>
        <td>1.0</td>
        <td>14.7</td>
        <td>1.0</td>
        <td>False</td>
        <td>132</td>
        <td>165</td>
        <td>1</td>
        <td>42.0</td>
        <td>0.5</td>
        <td>0.5</td>
        <td>8.65</td>
        <td>0.0</td>
        <td>0.3</td>
        <td>51.95</td>
        <td>0.0</td>
        <td>None</td>
    </tr>
</table>
<!-- #endregion -->

Hydra allows us to perform analytical queries efficiently. Let's compute the average trip distance given the passenger count:

```sql
SELECT
    passenger_count, AVG(trip_distance) AS avg_trip_distance
FROM taxi
GROUP BY passenger_count
ORDER BY passenger_count ASC
```

<!-- #region -->


**Console output (1/2):**

```txt
*  postgresql://user:***@localhost/db
10 rows affected.
```

**Console output (2/2):**

<table>
    <tr>
        <th>passenger_count</th>
        <th>avg_trip_distance</th>
    </tr>
    <tr>
        <td>0.0</td>
        <td>2.5424466811344758</td>
    </tr>
    <tr>
        <td>1.0</td>
        <td>2.6805563237138753</td>
    </tr>
    <tr>
        <td>2.0</td>
        <td>2.7948325921160876</td>
    </tr>
    <tr>
        <td>3.0</td>
        <td>2.7576410606577899</td>
    </tr>
    <tr>
        <td>4.0</td>
        <td>2.8681984015618327</td>
    </tr>
    <tr>
        <td>5.0</td>
        <td>2.6940995207308051</td>
    </tr>
    <tr>
        <td>6.0</td>
        <td>2.5745177825092658</td>
    </tr>
    <tr>
        <td>7.0</td>
        <td>11.1340000000000000</td>
    </tr>
    <tr>
        <td>8.0</td>
        <td>1.05000000000000000000</td>
    </tr>
    <tr>
        <td>None</td>
        <td>29.6651257727346673</td>
    </tr>
</table>
<!-- #endregion -->

JupySQL comes with powerful plotting capabilities. Let's create a histogram of trip distance:

```python
%sqlplot histogram --table taxi --column trip_distance --bins 50
```

<!-- #region -->


**Console output (1/2):**

```txt
<AxesSubplot: title={'center': "'trip_distance' from 'taxi'"}, xlabel='trip_distance', ylabel='Count'>
```

**Console output (2/2):**

![26-1](jupyter-images/26-1.png)
<!-- #endregion -->

We cannot see much since there are some outliers. Let's find the 99th percentile:

```sql
SELECT percentile_disc(0.99) WITHIN GROUP (ORDER BY trip_distance)
FROM taxi
```

<!-- #region -->


**Console output (1/2):**

```txt
*  postgresql://user:***@localhost/db
1 rows affected.
```

**Console output (2/2):**

<table>
    <tr>
        <th>percentile_disc</th>
    </tr>
    <tr>
        <td>19.24</td>
    </tr>
</table>
<!-- #endregion -->

Now, let's use this value as a cutoff:

```sql magic_args="--save no_outliers --no-execute"
SELECT trip_distance
FROM taxi
WHERE trip_distance < 19.24
```

<!-- #region -->


**Console output (1/1):**

```txt
*  postgresql://user:***@localhost/db
Skipping execution...
```

<!-- #endregion -->

```python
%sqlplot histogram --table no_outliers --column trip_distance --bins 50 --with no_outliers
```

<!-- #region -->


**Console output (1/2):**

```txt
<AxesSubplot: title={'center': "'trip_distance' from 'no_outliers'"}, xlabel='trip_distance', ylabel='Count'>
```

**Console output (2/2):**

![31-1](jupyter-images/31-1.png)
<!-- #endregion -->

Much better! We just created a histogram of 1.4M observations!


## Where to go from here

- [Hydra documenation](https://docs.hydras.io/)
- [JupySQL documentation](https://jupysql.readthedocs.io/en/latest/quick-start.html)


## Clean up

To finish the tutorial, let's shut down the container:

```python
! docker container ls
```

<!-- #region -->


**Console output (1/1):**

```txt
CONTAINER ID   IMAGE                    COMMAND                  CREATED         STATUS         PORTS                    NAMES
fd21cec1f520   ghcr.io/hydrasdb/hydra   "docker-entrypoint.s…"   9 minutes ago   Up 9 minutes   0.0.0.0:5432->5432/tcp   hydra
```

<!-- #endregion -->

```python
%%capture out
! docker container ls --filter ancestor=ghcr.io/hydrasdb/hydra --quiet
```

```python
container_id = out.stdout.strip()
print(f"Container id: {container_id}")
```

<!-- #region -->


**Console output (1/1):**

```txt
Container id: fd21cec1f520
```

<!-- #endregion -->

```python
! docker container stop {container_id}
```

<!-- #region -->


**Console output (1/1):**

```txt
fd21cec1f520
```

<!-- #endregion -->

```python
! docker container rm {container_id}
```

<!-- #region -->


**Console output (1/1):**

```txt
fd21cec1f520
```

<!-- #endregion -->

```python
! docker container ls
```

<!-- #region -->


**Console output (1/1):**

```txt
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

<!-- #endregion -->

## Package versions

For reproducibility, these are the package versions we used:

```python
# jupysql
import sql; sql.__version__
```

<!-- #region -->


**Console output (1/1):**

```txt
'0.5.3dev'
```

<!-- #endregion -->

```python
import pandas; pandas.__version__
```

<!-- #region -->


**Console output (1/1):**

```txt
'1.5.2'
```

<!-- #endregion -->

```python
import pyarrow; pyarrow.__version__
```

<!-- #region -->


**Console output (1/1):**

```txt
'8.0.0'
```

<!-- #endregion -->

```python
import psycopg2; psycopg2.__version__
```

<!-- #region -->


**Console output (1/1):**

```txt
'2.9.3 (dt dec pq3 ext lo64)'
```

<!-- #endregion -->

```python
import pgspecial; pgspecial.__version__
```

<!-- #region -->


**Console output (1/1):**

```txt
'1.13.1'
```

<!-- #endregion -->