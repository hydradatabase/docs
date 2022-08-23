---
description: Hydra is secured with end-to-end encryption with publicly-trusted certificates.
---

# 🔒 TLS

Hydra has publicly-trusted certificates from Let's Encrypt allowing you to connect safety to your data warehouse from anywhere on the Internet. To validate the certificate, you must use `sslmode=verify-full` when connecting and configure your Postgres connection to read your public certificate bundle.

If you are unable to configure your Postgres connection, we recommend using `sslmode=require`.&#x20;

## psql

### Locate your root certificate bundle

To configure psql, you will need to know the location of your root certificate bundle. On most systems, this file is located at `/etc/ssl/cert.pem`.&#x20;

If the file is not located in `/etc/ssl`, you can use `curl -v` to a secure URL and look for the `CAfile` line:

```shell-session
$ curl -v https://hydras.io/ 2>&1 | grep -i CAfile
*  CAfile: /etc/ssl/cert.pem
```

### Add sslrootcert to the connection string

The simplest option is to add the file to the end of the connection string using the parameter `sslrootcert`, as follows:

```
psql "postgres://.../d123456?sslmode=verify-full&sslrootcert=/etc/ssl/cert.pem"
```

### Add Hydra to you service file

You can manage and save your Hydra connection by creating an entry in your [service file](https://www.postgresql.org/docs/current/libpq-pgservice.html), located at `~/.pg_service.conf`.

<pre class="language-ini"><code class="lang-ini">[hydra]
user=u123456
password=UdW3zJT9FLfIpJrli47HMmL1
host=1ed22fba-b20a-6680-afd9-91fc7c62485e.us-east-1.aws.hydradb.io
<strong>port=5432
</strong><strong>dbname=d123456
</strong>sslmode=verify-full
<strong>sslrootcert=/etc/ssl/cert.pem</strong></code></pre>

Once you have added this entry, connect to your data warehouse using the name you specified at the top of block:

<pre><code><strong>psql service=hydra</strong></code></pre>

Any additional parameters will override your service entry. For example, you can use `psql service=hydra dbname=postgres` to connect to the `postgres` database.

### Always validate certificates

:warning: If you choose this path, `psql` will try to validate certificates when connecting to any Postgres database. This will cause connections to some other Postgres databases to return an error, even if you set `sslmode`.

You can instruct `psql` to always read your public certificate bundle my symlinking `~/.postgresql/root.crt` to your public root cert bundle:

```shell
mkdir -p ~/.postgresql
ln -s /etc/ssl/cert.pem ~/.postgres/root.crt
```

## From Your Application

You should refer to your application's Postgres library, but many libraries use `libpq` behind the scenes. You can configure `libpq` to read your cert bundle using the `PGSSLROOTCERT` environment variable. For example, add this environment variable to your application's environment:

```shell
PGSSLROOTCERT=/etc/ssl/cert.pem
```

Please double check the location of your CA cert bundle in your production environment.
