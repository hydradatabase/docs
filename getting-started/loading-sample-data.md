# Tutorial and Sample Data

This section contains sample dataset that you can load into Hydra.

## GitHub Events Dataset

We prepared a dataset from the [GH Archive](https://www.gharchive.org/) that contains 9MB of events on `2015-01-01-15` that have 4000 rows.
You can download it [here](../../.gitbook/assets/github_events.sql) and load it into Hydra:

```console
psql ... -f path_to_github_events.sql
```

or in `psql` console:

```
\d path_to_github_events.sql
```
