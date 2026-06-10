# 4.3 Testing and Verifying Nginx Stapling

## Validating Configuration Syntax

Before reloading your live Nginx processes, always execute an atomic configuration check to catch any syntax bugs:

```bash
sudo nginx -t

```

If the interface wrapper reports a successful verification trace state (`syntax is ok / test is successful`), apply the configuration changes:

```bash
sudo systemctl restart nginx

```

## Verifying Handshake Aggregation (Client Side)

From a remote Linux client workstation, run an external TLS handshake audit using the `openssl s_client` tool suite. The `-status` flag forces the connection handler to request and render stapled telemetry parameters:

```bash
openssl s_client -connect srv-web01.local:443 -status -TLSExtReqStatus

```

Parse the initial block outputs for the **"OCSP Response Data"** section:

### Expected Functional State

If the server is successfully caching and injecting the cryptographic token, you will see a validation trace like this:

```text
OCSP Response Data:
    OCSP Response Status: successful (0x0)
    Response Type: Basic OCSP Response
    Cert Status: good
    This Update: Jun  4 20:00:00 2026 GMT
    Next Update: Jun  5 20:00:00 2026 GMT

```

### Inactive or Failing State

If the integration setup fails or is disabled, the connection handler outputs:

```text
OCSP response: no response sent

```

> 💡 **The Sysadmin Trap:** When Nginx is first restarted, it may return `no response sent` on the very first connection attempt. This behavior is normal: Nginx fetches the OCSP validation token asynchronously *after* encountering the initial inbound connection. Run the verification command a second time to ensure the handshake token is properly stapled.

