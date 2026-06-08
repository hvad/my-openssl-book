# 1.4 Useful Verification Commands

As a Systems Engineer, you will use these fundamental diagnostic verification commands 
most of the time to troubleshoot and audit cryptographic assets:

### Verify a Certificate Signing Request (CSR)
Inspect the identity fields, validation parameters, and extensions inside an uncompiled request file:
```bash
openssl req -text -noout -verify -in csr/web.local.csr.pem

```

### Verify an X.509 Public Certificate

Inspect the compiled public properties, validity dates, serial numbers, and extensions of an active certificate asset:

```bash
openssl x509 -text -noout -in certs/web.local.cert.pem

```

### Key and Certificate Cryptographic Pair Validation (Modulus Match)

To verify that a public certificate corresponds exactly to a specific private key, extract and compare the 
cryptographic MD5 hash of their mathematical modulus. If both hash outputs match identically, the key pair is validated:

```bash
openssl x509 -noout -modulus -in certs/web.local.cert.pem | openssl md5
openssl rsa -noout -modulus -in private/web.local.key.pem | openssl md5

```

### Verify the Local Trust Chain Validity

Validate that an end-entity leaf server certificate is signed and properly cryptographically authorized by the upstream Intermediate CA:

```bash
openssl verify -CAfile certs/intermediate.cert.pem certs/web.local.cert.pem

```
