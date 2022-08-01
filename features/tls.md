# TLS

Hydra has valid, publically signed certificates from Let's Encrypt allowing you to connect safety to your data warehouse from anywhere on the Internet. To validate the certificate, you must use `sslmode=verify-full` when connecting and configure your Postgres connection to read your public certificate bundle.

If you are unable to configure your Postgres connection, we recommend using `sslmode=require`.&#x20;

### psql

Instruct `psql` to read your public certificate bundle my symlinking `~/.postgresql/root.crt` to your public root cert bundle, typically at `/etc/ssl/cert.pem`:

```shell
mkdir -p ~/.postgresql
ln -s /etc/ssl/cert.pem ~/.postgres/root.crt
```

The location of `/etc/ssl/cert.pem` may vary on your system.

### add From Your Application

You should refer to your application's Postgres library, but many libraries use `libpq` behind the scenes. You can configure `libpq` to read your cert bundle using the `PGSSLROOTCERT` environment variable. For example, add this environment variable to your application's environment:

```shell
PGSSLROOTCERT=/etc/ssl/cert.pem
```

Please double check the location of your CA cert bundle in your production environment.
