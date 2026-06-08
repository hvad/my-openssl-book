# 2.3 Testing & Verifying Revocation

Once revocation configurations and services are live, you can execute client validation queries to audit trust state boundaries.

### Validating Against a Local CRL File

To test if a target certificate is flagged on a downloaded CRL blacklist manifest, pass the full authoritative structure to the OpenSSL verification subsystem:

```bash
openssl verify -CAfile certs/intermediate.cert.pem \
               -CRLfile crl/intermediate.crl \
               -crl_check certs/web.local.cert.pem

```

*If the targeted certificate asset has been successfully revoked, the command returns an explicit failure trace statement:*
`error 23 at 0 depth lookup: certificate revoked`

### Validating via Real-Time OCSP Queries

While the test OCSP daemon daemon processes requests on port `2560`, simulate an external browser client lookup from a separate terminal interface wrapper:

```bash
openssl ocsp -issuer certs/intermediate.cert.pem \
             -cert certs/web.local.cert.pem \
             -url [http://127.0.0.1:2560](http://127.0.0.1:2560) \
             -CAfile certs/intermediate.cert.pem

```

The terminal interface will parse and print the standard cryptographic trace verification envelope payload:

* Successful target: `certs/web.local.cert.pem: good`
* Invalidated target: `certs/web.local.cert.pem: revoked`

