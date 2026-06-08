# 1.2 Intermediate CA Creation

The Intermediate CA is responsible for executing day-to-day certificate signing tasks. 
This architecture keeps your Root CA protected offline while the Intermediate CA operates online or semi-online.

## Directory Initialization

Create a separate, dedicated workspace for the intermediate tier:

```bash
mkdir -p ~/pki/intermediate/{certs,crl,csr,newcerts,private}
touch ~/pki/intermediate/index.txt
echo 1000 > ~/pki/intermediate/serial
chmod 700 ~/pki/intermediate/private
cd ~/pki/intermediate

```

## Generate the Intermediate Private Key

Generate an encrypted 4096-bit RSA private key for the intermediate layer:

```bash
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -aes256 -out private/intermediate.key.pem

```

## Create the Certificate Signing Request (CSR)

Instead of creating a self-signed certificate, the intermediate authority must generate a CSR to seek endorsement from the Root CA tier:

```bash
openssl req -config /etc/ssl/openssl.cnf \
      -key private/intermediate.key.pem \
      -new -sha256 -out csr/intermediate.csr.pem

```

## Sign the CSR with the Root CA

Navigate back into your Root CA operational directory (`cd ~/pki/root-ca`) to sign the incoming request. We append the `v3_intermediate_ca` extension block to explicitly declare its certificate-signing privileges:

```bash
openssl ca -config /etc/ssl/openssl.cnf \
      -extensions v3_intermediate_ca -days 3650 -notext -md sha256 \
      -in ../intermediate/csr/intermediate.csr.pem \
      -out ../intermediate/certs/intermediate.cert.pem

```
