## Summary of SSL Creation Process

| Step                                          | Command                                                                                                 | Purpose                                                                                                                     |
|-----------------------------------------------|---------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------|
| CA Creation                                   |                                                                                                         |                                                                                                                             |
| 1. Generate CA Private Key                    | openssl genpkey -algorithm RSA -out ca.key                                                              | Creates a secure RSA private key for the CA, used for signing certificates.                                                 |
| 2. Generate CA Certificate                    | openssl req -new -x509 -days 365 -key ca.key -out ca.crt -subj ""/CN=MyCA""                             | Creates a self-signed certificate for the CA that includes the CA's public key, establishing its identity.                  |
| Server Certificate Creation                   |                                                                                                         |                                                                                                                             |
| 1. Generate Server Private Key                | openssl genpkey -algorithm RSA -out server.key                                                          | Creates a secure RSA private key for the server, used for encrypting data and establishing a secure connection.             |
| 2. Create a Certificate Signing Request (CSR) | openssl req -new -key server.key -out server.csr -subj ""/CN=192.168.113.133""                          | Generates a CSR that includes the server's public key and identifying information (such as the server's IP or hostname).    |
| 3. Sign the CSR with CA Certificate           | openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365     | Signs the server's CSR with the CA's private key, creating a server certificate (server.crt) that is trusted by clients.    |
| 4. Configure Server to Use the Certificate    | Configure PostgreSQL to use server.crt and server.key files in the configuration.                       | Enables SSL on the server, allowing secure connections with clients using the signed server certificate.                    |
| Client Certificate Creation                   |                                                                                                         |                                                                                                                             |
| 1. Generate Client Private Key                | openssl genpkey -algorithm RSA -out client.key                                                          | Creates a secure RSA private key for the client, used for encrypting data and establishing a secure connection.             |
| 2. Create a Certificate Signing Request (CSR) | openssl req -new -key client.key -out client.csr -subj ""/CN=postgres""                                 | Generates a CSR that includes the client’s public key and identifying information (such as the client username).            |
| 3. Sign the CSR with CA Certificate           | openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 365     | Signs the client's CSR with the CA's private key, creating a client certificate (client.crt) that is trusted by the server. |
| 4. Configure Client to Use the Certificate    | Transfer client.crt, client.key, and ca.crt to the client machine and configure the client to use them. | Enables the client to authenticate securely to the server using the signed client certificate during the SSL handshake.     |


### components of SSL 

| Component                       | Description                                                                                                                                                                                   |   
|---------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| CA Private Key (ca.key)         | The private key of the Certificate Authority, used to sign certificates (server and client) to establish their authenticity.                                                                  |   
| CA Certificate (ca.crt)         | The public certificate of the CA, which contains the CA's public key. It is used to verify the signatures of the server and client certificates.                                              |   
| Server Private Key (server.key) | The private key for the server, used to decrypt messages encrypted with the server's public key and to establish secure sessions.                                                             |   
| Server Certificate (server.crt) | The server's public key certificate, which includes the server's public key. It is signed by the CA and allows clients to verify the server's identity.                                       |   
| Client Private Key (client.key) | The private key for the client, used to decrypt messages encrypted with the client’s public key and to establish secure sessions.                                                             |   
| Client Certificate (client.crt) | The client’s public key certificate, which includes the client’s public key. It is signed by the CA and allows the server to verify the client’s identity (if mutual authentication is used). |   

### SSL modes

| sslmode     | SSL Required | Certificate Validation | Hostname Validation |
|-------------|--------------|------------------------|---------------------|
| disable     | No           | No                     | No                  |
| allow       | No           | No                     | No                  |
| prefer      | No           | No                     | No                  |
| require     | Yes          | No                     | No                  |
| verify-ca   | Yes          | Yes                    | No                  |
| verify-full | Yes          | Yes                    | Yes                 |
