# 1.3 Server Certificate Generation

With our infrastructure established, we can now assume the administrative role of issuing 
an end-entity leaf certificate for an operational application tier, such as an Nginx server (`web.local`).

## Generate the Server Private Key

> **Admin Tip:** Web server private keys should **not** be encrypted with an interactive passphrase. If encrypted, web daemons like Nginx or Apache HTTPD will hang during an unattended server reboot, waiting for manual human entry of the decryption credential.

```bash
cd ~/pki/intermediate
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out private/web.local.key.pem

```

## Create the CSR with Subject Alternative Names (SAN)

Modern client user agents and browsers (such as Google Chrome) strictly require the **Subject Alternative Name (SAN)** 
extension to prevent hostname mismatch errors. To specify these parameters, create a temporary configuration manifest file named `san.cnf`:

```ini
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
C = FR
O = Mon Entreprise
CN = web.local

[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = web.local
DNS.2 = www.web.local
IP.1 = 192.168.1.50

```

Generate the CSR using this localized configuration definition:

```bash
openssl req -new -out csr/web.local.csr.pem \
      -key private/web.local.key.pem \
      -config san.cnf

```

## Sign the Leaf Certificate with the Intermediate CA

Use the Intermediate CA database to process and sign the server certificate. By standard operational 
compliance rules, this is bound to a shorter validity cycle (e.g., 1 year / 365 days):

```bash
openssl ca -config /etc/ssl/openssl.cnf \
      -extensions server_cert -days 365 -notext -md sha256 \
      -in csr/web.local.csr.pem \
      -out certs/web.local.cert.pem

```
