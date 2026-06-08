# 2.2 Online Certificate Status Protocol (OCSP)

The Online Certificate Status Protocol (OCSP) replaces standard CRL files with a real-time HTTP query API framework. Instead of downloading a massive blacklist, client agents can query the status of a specific certificate serial number.

## Step A: Configure the CA to Embed the OCSP Endpoint

To declare the validation responder network coordinate, modify your Intermediate CA configuration profile block (`[ server_cert ]`) before signing new production certificate payloads:

```ini
[ server_cert ]
# ... surrounding operational configurations ...
authorityInfoAccess = OCSP;URI:[http://ocsp.monentreprise.local:2560](http://ocsp.monentreprise.local:2560)

```

## Step B: Provisioning a Dedicated OCSP Signing Keypair

For security containment, the primary Intermediate CA private key should not be used to sign transactional OCSP requests continuously. Instead, generate a dedicated cryptographic operational keypair with an explicit short lifecycle:

```bash
cd ~/pki/intermediate
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out private/ocsp.key.pem

```

Create a Certificate Signing Request (CSR) for the responder identity:

```bash
openssl req -config /etc/ssl/openssl.cnf \
      -new -sha256 \
      -key private/ocsp.key.pem \
      -out csr/ocsp.csr.pem

```

Sign the request using your upstream database, attaching the specialized `ocsp` application profile extension flag:

```bash
openssl ca -config /etc/ssl/openssl.cnf \
      -extensions ocsp -days 365 -notext -md sha256 \
      -in csr/ocsp.csr.pem \
      -out certs/ocsp.cert.pem

```

## Step C: Launching the OpenSSL Built-In OCSP Daemon

For development testing, automation orchestration validation, or internal lab use, OpenSSL includes a lightweight, built-in HTTP server capability to answer status queries on port `2560`:

```bash
openssl ocsp -index index.txt \
      -port 2560 \
      -text \
      -rsigner certs/ocsp.cert.pem \
      -rkey private/ocsp.key.pem \
      -CA certs/intermediate.cert.pem

```
