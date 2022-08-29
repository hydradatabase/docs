---
description: >-
  Hydra is a data warehouse that is accessible from Postgres. You can start
  small with Postgres, then switch to the data engine of your choice at any
  time.
---

# 💡 What is Hydra?

All Hydra instances include:

* Encrypted connections and data stores
* 30 days backups with point-in-time recovery
* Seamlessly scale up storage

## Quick Start

Begin with a Hydra data warehouse, then use standard Postgres tools to copy data. You'll need to substitute the credentials as appropriate. Each command below will prompt for the password.

```shell
pg_dump -Fc --no-acl --no-owner \
  -h your.db.hostname \
  -U username \
  databasename > mydb.dump
  
pg_restore --verbose --clean --no-acl --no-owner \
  -h 623942e76454.db.hydras.io \
  -U uyuc0wnp2zmp \
  -d dida6lgi5dpl \
   mydb.dump
```

Once you have your data imported, you can start using your Hydra data warehouse with any PostgreSQL-compatible tools and adapters.

{% hint style="info" %}
Your Hydra username always begins with `u`, and your database always begins with `d`.
{% endhint %}
