# NOTES

## Links

- [Radical Candor In Practice — Sharebold](https://sharebold.com/posts/a-curious-tale-of-rust-tls-and-postgres-in-the-cloud-434k)
- [Getting Title at 50:23](https://github.com/ecliptical/tokio-postgres-rustls-rds-demo)
- [How to Setup PostgreSQL with SSL inside a Docker Container](https://dev.to/danvixent/how-to-setup-postgresql-with-ssl-inside-a-docker-container-5f3)
- [Enabling SSL for PostgreSQL in Docker](https://gist.github.com/mrw34/c97bb03ea1054afb551886ffc8b63c3b)

```shell
$ env PG.DBNAME=postgres PG.HOST=localhost PG.PORT=6432 PG.USER=postgres PG.PASSWORD=postgres DB_CA_CERT=docker/postgres-certs/ca.pem RUST_LOG=debug cargo run
```

> OPTED : to use certificates that comes with image (ssl-cert-snakeoil.pem), else with self signed or gererated certificates we always have permissions problems if use outside certificates, even with same permissions

```shell
$ docker run \
  -it \
  --rm \
  -e POSTGRES_PASSWORD=password \
  -p 6432:5432 \
  postgres:12.2 \
  -c ssl=on \
  -c ssl_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem \
  -c ssl_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
# outcome
2022-08-23 21:21:51.975 UTC [1] LOG:  database system is ready to accept connections

# using ssl-cert-snakeoil.pem from inside container with localhost will fail
$ env PG.DBNAME=postgres PG.HOST=localhost PG.PORT=6432 PG.USER=postgres PG.PASSWORD=postgres DB_CA_CERT=$(pwd)/ssl-cert-snakeoil.pem RUST_LOG=debug cargo run
# outcome
2022-08-17 00:58:42.612 UTC [64] LOG:  could not accept SSL connection: sslv3 alert bad certificate

# get docker container id
$ docker ps | grep 6432
fa636caa65ab   postgres:12.2                    "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes    0.0.0.0:6432->5432/tcp, :::6432->5432/tcp                                                         sad_lalande
# alert bad certificate SOLUTION on inspect certificate get the hostname from CN ex b9bdf96ee5bd
$ docker exec -it fa636caa65ab bash
$ openssl x509 -in /etc/ssl/certs/ssl-cert-snakeoil.pem -text -noout | grep "Subject: CN"
        Subject: CN = b9bdf96ee5bd
# add it to hosts
127.0.0.1       b9bdf96ee5bd

# bring certificate
$ docker cp fa636caa65ab:/etc/ssl/certs/ssl-cert-snakeoil.pem .

# and use it and now certificates works
$ HOST=b9bdf96ee5bd
$ env PG.DBNAME=postgres PG.HOST=${HOST} PG.PORT=6432 PG.USER=postgres PG.PASSWORD=password DB_CA_CERT=$(pwd)/ssl-cert-snakeoil.pem RUST_LOG=debug cargo run
# outcome
 DEBUG rustls::anchors                            > add_pem_file processed 1 valid and 0 invalid certs
 DEBUG rustls::client::hs                         > No cached session for DNSNameRef("b9bdf96ee5bd")
 DEBUG rustls::client::hs                         > Not resuming any session
 DEBUG rustls::client::hs                         > No cached session for DNSNameRef("b9bdf96ee5bd")
 DEBUG rustls::client::hs                         > Not resuming any session
 DEBUG rustls::client::hs                         > Using ciphersuite TLS13_AES_256_GCM_SHA384
 DEBUG rustls::client::tls13                      > Not resuming
 DEBUG rustls::client::tls13                      > TLS1.3 encrypted extensions: []
 DEBUG rustls::client::hs                         > ALPN protocol is None
 DEBUG rustls::client::tls13                      > Ticket saved
 DEBUG rustls::client::tls13                      > Ticket saved
 DEBUG tokio_postgres::prepare                    > preparing query s0: SELECT * FROM information_schema.information_schema_catalog_name
 DEBUG tokio_postgres::query                      > executing statement s0 with parameters: []
 INFO  tokio_postgres_rustls_connection_pool_demo > postgres

now it works
```
