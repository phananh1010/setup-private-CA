# Introduction
A Private CA can serve as a straightforward authentication scheme to ensure secure communication between computer nodes in a network. In this repository, I present a scenario where a private CA is utilized to facilitate authentication in the context of Federated Learning. The FLServer will request a signed certificate from the PrivateCA. Each time the FLClient connects to the FLServer, the FLClient will verify the certificate presented by the FLServer with the PrivateCA.

                  PrivateCA
                /           \
               /             \
              /               \
             /                 \
      FLClient <-------------> FLServer


In the following, we discuss the set-up of the private Certificate Authority (CA) infrastructure, allowing authentication between Federated Learning (FL) Server and clients.

# Step-by-step Instructions
## 1. Setting-up a private CA
Private CA will be a stand alone machine with self-signed certificate. The private CA will receive CSR request, which is a file, sent from a FL server and generate a signed certificate. The certificate will be sent to the FL server and all FL clients.

#### Generate private key for private CA
```
openssl genrsa -out ca.key 2048
```
#### Generate self-signed certificate for private CA
```
openssl req -new -x509 -key ca.key -out ca.crt
```

## 2 .Creating CSR request from FL Server
The server will generate a CSR request and sent the CSR file to the server. The server must specify a configuration file including necessary information to generate CSR file. Here is a content of the configuration file:
```
[req]
default_bits = 2048
encrypt_key = no
default_keyfile = FLServer.key
default_md = sha1
prompt = no
req_extensions = req_ext
distinguished_name = dn

[dn]
C = DE
ST = HH
O = Flower
CN = localhost

[req_ext]
# won't use server cert to sign other cert (not CA)
basicConstraints=CA:FALSE
subjectAltName = @alt_names
subjectKeyIdentifier = hash

[alt_names]
DNS.1 = localhost
IP.1 = ::1
IP.2 = 127.0.0.1
```

Generate CSR file based on the configure file:
```
openssl req -new -out FLServer.csr -config certificate.conf 
```

Here are some important information in the config file:
- Default bit key is 2048: ```default_bits = 2048```
- Specify the name of the private key for FL server. A new key will be also be created after CSR file is generated. ```default_keyfile = FLServer.key```
- Prevent server cert to be used to sign other cert: ```basicConstraints=CA:FALSE```
- In testing environment, set the IP and DNS of FL server to be local host. ```DNS.1 = localhost``` and ```IP.2 = 127.0.0.1```

## 3. Generating certificate for FL Server from private CA
Upon receiving the CSR file, the private CA generate the certificate for FL server.
```
openssl x509 -req -in FLServer.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out FLServer.crt
```

## 4. Note on secure-comminication between FL Server, private CA, and FL Clients to exchange CSR files and certificate
  - The FL server can send the CSR file to the private CA via secured channel such as via https, email, SFTP
  - The private CA can send the certificate to the FL server and FL client via secured channel such as email, https, SFTP
    
