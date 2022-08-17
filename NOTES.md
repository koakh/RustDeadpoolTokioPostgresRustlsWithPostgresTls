# NOTES

```shell
$ env PG.DBNAME=postgres PG.HOST=localhost PG.PORT=6432 PG.USER=postgres PG.PASSWORD=postgres DB_CA_CERT=docker/postgres-certs/ca.pem RUST_LOG=debug cargo run
```

used certificates that comes with image, else we always have permissions propblems if use outside certificates, even with same permissions

```shell
docker run \
  -it \
  --rm \
  -e POSTGRES_PASSWORD=password \
  -p 6432:5432 \
  postgres:12.2 \
  -c ssl=on \
  -c ssl_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem \
  -c ssl_key_file=/etc/ssl/private/ssl-cert-snakeoil.key

# using ssl-cert-snakeoil.pem from inside container
$ env PG.DBNAME=postgres PG.HOST=localhost PG.PORT=6432 PG.USER=postgres PG.PASSWORD=postgres DB_CA_CERT=$(pwd)/ssl-cert-snakeoil.pem RUST_LOG=debug cargo run

# outcome
2022-08-17 00:58:42.612 UTC [64] LOG:  could not accept SSL connection: sslv3 alert bad certificate

# SOLUTION on inspect certificate get the hostname from CN ex b9bdf96ee5bd
$ openssl x509 -in ssl-cert-snakeoil.pem -text -noout | grep "Subject: CN"
        Subject: CN = b9bdf96ee5bd

# add it to hosts
127.0.0.1       b9bdf96ee5bd

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
