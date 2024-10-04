```
# Generate the CA private key
openssl genpkey -algorithm RSA -out ca.key

# Generate the CA certificate
openssl req -new -x509 -days 365 -key ca.key -out ca.crt -subj "/CN=MyCA"

#Create the Server Certificate
openssl genpkey -algorithm RSA -out server.key
openssl req -new -key server.key -out server.csr -subj "/CN=192.168.113.133"
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365
chmod 600 server.key

#Create the Client Certificate
openssl genpkey -algorithm RSA -out client.key
openssl req -new -key client.key -out client.csr -subj "/CN=postgres"
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 365
chmod 600 client.key

## copy files to respective locations
cp server.crt server.key ca.crt /u01/pgsql/17
chmod 600 /u01/pgsql/17/server.key

## Edit postgresql.conf and pg_hba.conf

ssl = on
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'
ssl_ca_file = 'ca.crt'

hostssl    all    all    0.0.0.0/0    md5 clientcert=verify-full
hostssl    all    all    0.0.0.0/0    cert 

## Make changes in client side
mkdir ~/.postgresql
scp client.crt client.key ca.crt 192.168.113.137:/home/postgres.postgresql/
chmod 600 /home/postgres/.postgresql/client.key

psql "host=192.168.113.133 dbname=postgres user=postgres sslmode=verify-full sslcert=client.crt sslkey=client.key sslrootcert=ca.crt"
```
