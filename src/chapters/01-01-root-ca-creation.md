# 1.1 Root CA Creation

The Root CA represents the absolute trust anchor for your infrastructure. 
It must be isolated in its own dedicated workspace directory with strict filesystem permissions.

## Directory Initialization

Run the following commands to create the environment layout and initialize the tracking database components:

```bash
mkdir -p ~/pki/root-ca/{certs,crl,newcerts,private}
touch ~/pki/root-ca/index.txt
echo 1000 > ~/pki/root-ca/serial
chmod 700 ~/pki/root-ca/private
cd ~/pki/root-ca

```

## Generate the Root Private Key

> **Security Warning:** A Root CA private key must **always** be protected by a strong, high-entropy passphrase.

Generate an encrypted 4096-bit RSA private key using AES-256:

```bash
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -aes256 -out private/ca.key.pem

```

## Generate the Self-Signed Root Certificate

Because this is the root of the hierarchy, the certificate is self-signed. It is typically configured 
with a long operational lifespan (e.g., 10 years / 3650 days):

```bash
openssl req -config /etc/ssl/openssl.cnf \
      -key private/ca.key.pem \
      -new -x509 -days 3650 -sha256 -extensions v3_ca \
      -out certs/ca.cert.pem

```
