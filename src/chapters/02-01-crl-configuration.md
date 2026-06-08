# 2.1 Certificate Revocation Lists (CRL)

A Certificate Revocation List (CRL) acts as a cryptographic blacklist distributed by 
the Certificate Authority. For client agents to locate and process this data, 
distribution anchors must be injected directly into the end-entity certificate extensions during execution.

## Step A: Configure the CA to Embed the CRL URL

Open your Intermediate CA OpenSSL configuration database blueprint file 
(commonly `/etc/ssl/openssl.cnf` or a localized configuration manifest) and 
map the `crlDistributionPoints` parameter within the active leaf server profile block (`[ server_cert ]`):

```ini
[ server_cert ]
# ... surrounding operational configurations ...
crlDistributionPoints = URI:[http://pki.monentreprise.local/crl/intermediate.crl](http://pki.monentreprise.local/crl/intermediate.crl)

```

## Step B: Executing a Certificate Revocation Command

If a server private key is leaked or compromised, you must instantly disqualify 
the asset using the Intermediate CA execution toolkit:

```bash
cd ~/pki/intermediate
openssl ca -config /etc/ssl/openssl.cnf \
      -revoke certs/web.local.cert.pem \
      -cReason keyCompromise

```

> 💡 **Admin Tip:** Valid options for the revocation context argument (`-cReason`) include: `keyCompromise`, `CACompromise`, `affiliationChanged`, `superseded`, `cessationOfOperation`, or `certificateHold`.

## Step C: Compiling and Publishing the Updated CRL

Revoking a certificate changes the internal text tracking database (`index.txt`), but 
it does **not** update the public file. You must explicitly generate a signed binary or PEM 
format CRL file to publish it to your distribution node servers:

```bash
openssl ca -config /etc/ssl/openssl.cnf \
      -gencrl -out crl/intermediate.crl

```

