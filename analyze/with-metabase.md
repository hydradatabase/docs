---
description: Business intelligence tooling
---

# With Metabase

As [Metabase](https://www.metabase.com/) puts it:

> **Business intelligence for everyone** Team doesn’t speak SQL? No problem. Metabase is the easy, open-source way to help everyone in your company work with data like an analyst.

## Setup

- Ensure you have the details of your Hydra instance handy.
- Sign up for [Metabase](https://www.metabase.com/).

### Seed some sample data

Let’s start by creating a sample table in Hydra and populate it with some data. You'll need to replace your Hydra connection string or set the `PGCONN` environment variable.

* Create a sample table and seed some data.

    ```
    psql "$PGCONN" \
      -c "CREATE TABLE sample_data (id uuid NOT NULL PRIMARY KEY DEFAULT gen_random_uuid(), sample integer, timestamp timestamptz) USING columnar;" \
      -c "INSERT INTO sample_data (sample, timestamp) SELECT floor((random() + random() + random() + random() + random() + random()) / 6 * 100000000)::int, to_timestamp(EXTRACT(epoch from NOW()) - floor(random() * 2600000)) FROM generate_series(1, 50000);"
    ```

We’ve created a table named `sample_data` with 50,000 random samples spread out over the last month.

#### Metabase first launch

Now we can configure Metabase to access our sample data in the Hydra.

* Head over to Metabase and log in.
* Add your Hydra instance as outlined in the below screenshot.

![Metabase, add your data](https://hydras.io/assets/blog/2022-09-28/step\_3-15d9ce64e2c477490120b06c399d023b2584aaa8fba44bf35a1a5e166914051d.jpg)

* Once you’ve landed at the Metabase dashboard you should see some options to view insights after a few seconds. Take some time to explore Metabase, for example “a glance at Data”

### Metabase: Queries, Questions and Dashboards

Now let’s combine the features of Metabase and Hydra to explore our sample data.

#### Queries

You can use Metabase to run ad hoc queries against Hydra. You can access it via New → SQL query.

![Howdy, Hydra](https://hydras.io/assets/blog/2022-09-28/howdy-2d387a21fde79ff7c3d899779e26ca46fb25a66cee814d2857cca51138dd0d4b.png)

```
SELECT
    MIN(sample), MAX(sample), COUNT(sample), AVG(sample), STDDEV(sample)
FROM sample_data;
```

For example we can run some summary statistics against our sample data using:

![example question](https://hydras.io/assets/blog/2022-09-28/question-6bffdbf78d1b3f2a2b31097b0f58d020337a4d93950f8803b405ed3e9d43b743.png)

#### Questions

Metabase allows you to use questions to get answers from your data. Let’s create some questions now, that we’ll later turn into a dashboard. You can create questions via New → Question.

Let’s create four questions:

1.  Number of samples over the last 7 days

    ![number of samples over the last 7 days settings](https://hydras.io/assets/blog/2022-09-28/num\_samples\_settings-148fff4dc06553130c979b6fab27a81c3efedffde53594f76b7cf6abb9c5be73.png)

    ![number of samples over the last 7 days example](https://hydras.io/assets/blog/2022-09-28/num\_samples-ca80035bc8ff1d6f9f6a73a4d51c13cfc0f2d4b5ec3a0d2deb17d2750063b19f.png)
2.  Average values of samples over the last 7 days

    ![average values over the last 7 days settings](https://hydras.io/assets/blog/2022-09-28/avg\_value\_7\_settings-c0a2ea0331de6d46e08b9e34476876dcc5ee436ff162fc2a8848aecdc725dd03.png)

    ![average values over the last 7 days example](https://hydras.io/assets/blog/2022-09-28/avg\_value\_7-afdb7eab1a20d683c0e858fb57543b1eac90f0b9e17d485ee4e1ce31f803c438.png)
3.  Average value of samples per day over the last 30 days

    ![average values over the last 30 days settings](https://hydras.io/assets/blog/2022-09-28/avg\_value\_30\_settings-f877b59915a6410dc60830a95eb2beb7145a9aef50d962944b60833a2047cefd.png)

    ![average values over the last 30 days example](https://hydras.io/assets/blog/2022-09-28/avg\_value\_30-f5e37cb9e1c9a8e3bc4581c973fd79e87152aaa42b4118557e4d9637da0ff3b5.png)
4.  Distribution of Samples over the last 30 days

    ![distribution over the last 30 days settings](https://hydras.io/assets/blog/2022-09-28/distribution\_settings-cb1ef864847558a7a0f96b539714ed8c0e6eaf397ef7ae31dbce93b396455ab4.png)

    ![distribution over the last 30 days example](https://hydras.io/assets/blog/2022-09-28/distribution-a043d40a65a1b237fa4999542e0528d3b03f8a09ac73d9201266eab8f4dc848a.png)

#### Dashboards

Now let’s combine those questions into a dashboard to give our users at a glance access to these metrics.

1. Create a new dashboard via New → Dashboard
2. Add the four questions created above to the dashboard

![dashboard setup](https://hydras.io/assets/blog/2022-09-28/dashboard\_setup-6f0e9158725eb3a1f5e0189febd6888bb9f58fe1e103359a296c306c9b0fdd8b.png)

![dashboard example](https://hydras.io/assets/blog/2022-09-28/dashboard-b08d4b2f496e31566c6ae8bfa207139fc62dfd16ec6deae6b98ba0ed651f7adf.png)

#### Cleanup

Once you’re done exploring Metabase and Hydra you can cleanup the sample data be droping the `sample_data` table.

```
psql "$PGCONN" -c "DROP TABLE sample_data;"
```
