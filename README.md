# Connecting Securely for PostgreSQL amd using deadpool connection pooler

This project demonstrates how to use [Tokio Postgres](https://crates.io/crates/tokio-postgres) with [Rustls](https://crates.io/crates/rustls) to connect to PostgreSQL over TLS.

## Influenced repositories

- <https://github.com/ZPascal/tokio-postgres-rustls-connection-pool-demo.git>
- <https://github.com/ecliptical/tokio-postgres-rustls-rds-demo>

## TLDR

```bash
# first add certificate hostname to hosts, we need to connect to postgres with this hostname for TLS works
$ sudo nano /etc/hosts
127.0.0.1       b9bdf96ee5bd

# run with docker run
$ docker run \
  -it \
  --rm \
  -e POSTGRES_PASSWORD=password \
  -p 6432:5432 \
  --name postgres-rust \
  postgres:12.2 \
  -c ssl=on \
  -c ssl_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem \
  -c ssl_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
# outcome
2022-08-23 21:21:51.975 UTC [1] LOG:  database system is ready to accept connections

# or run with docker compose
$ cd docker && sudo rm docker/data/ -r && docker-compose up

# Don't change this host, this is a requirement fot TLS to work, this is the hostname of the ssl-cert-snakeoil.pem certificate, check note
$ HOST="b9bdf96ee5bd"
$ DB_CA_CERT="$(pwd)/ssl-cert-snakeoil.pem"
# run with tls
$ env PG.HOST=${HOST} DB_CA_CERT=${DB_CA_CERT} PG.DBNAME=postgres PG.PORT=6432 PG.USER=postgres PG.PASSWORD=password PG.SSL_MODE=Require RUST_LOG=debug cargo run

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
```
