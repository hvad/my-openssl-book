# 1.5 Deployment & Full Chain Files

To prevent validation anomalies, handshake negotiation failures, or trust errors across diverse 
client user agents (such as `curl` or client web browsers), web application servers must serve 
the end-entity certificate **and** the authoritative Intermediate CA certificate simultaneously during the TLS handshake.

To achieve this, you must compile an aggregated cryptographic file known as a **Full Chain Bundle**:

```bash
cat certs/web.local.cert.pem certs/intermediate.cert.pem > certs/web.local.fullchain.pem

```

### Web Daemon Reference Configuration

When setting up your reverse-proxies or edge web nodes, map your directives to utilize the compiled bundle and the unencrypted private key:

* **Nginx Parameter Mapping Example:**
```nginx
ssl_certificate     /root/pki/intermediate/certs/web.local.fullchain.pem;
ssl_certificate_key /root/pki/intermediate/private/web.local.key.pem;

```

### Client Trust Provisioning

For corporate workstations or internal Linux clients to inherently trust certificates issued by 
this custom PKI, distribute the **Root CA certificate** (`ca.cert.pem`) into their system-wide trusted storage engines:

```bash
sudo cp ca.cert.pem /usr/local/share/ca-certificates/my-root-ca.crt
sudo update-ca-certificates

```
