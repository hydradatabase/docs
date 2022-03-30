# Airbyte

Airbyte can be used to import and update data into your Hydra data warehouse from a [wide variety of sources](https://docs.airbyte.com/integrations/sources). You can choose how often Airbyte should update your data in Hydra, from one-time, weekly, daily, or hourly.

### Setup

Setting up Airbyte is easy and takes only a few minutes.

1. Create an Airbyte account at [https://cloud.airbyte.io/signup](https://cloud.airbyte.io/signup).&#x20;
2. Add Hydra as a "Destination."
   1. If using Hydra with Postgres, use a "Postgres" destination.
   2. If using Hydra with another engine, you can either use "Postgres" or your current engine for a direct connection. We recommend Postgres for ongoing changes, but use a direct connection for large one-time imports. \
      :information\_source: If making a direct connection, ask Hydra staff for the needed connection details.
3. Add your existing data sources as a "Source." Airbyte supports a [long list of sources](https://docs.airbyte.com/integrations/sources), including SaaS providers and databases.

### Syncing Data

Create a "Connection" for each Source you wish to connect to Hydra. Choose what data you want to sync, at what frequency, and what [sync mode](https://docs.airbyte.com/understanding-airbyte/connections) to use. Make sure to enable Airbyte's [basic normalization feature](https://docs.airbyte.com/understanding-airbyte/basic-normalization).

### Recommendations

* When possible, design data coming into the data warehouse as append-only. Columnar data warehouse engines are very slow when updating or deleting data.
* Schema changes are difficult when a data source is connected through Airbyte. When possible, connect and insert data into Hydra directly after completing an initial import with Airbyte.
* If you are uncertain how to design your data warehouse, reach out to Hydra for advice.
