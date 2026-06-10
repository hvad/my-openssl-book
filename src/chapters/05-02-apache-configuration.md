# 5.2 Apache VirtualHost Configuration

Below is the hardened virtual host configuration layout profile optimized for port `443` execution on Red Hat topologies. 

On RHEL/Fedora systems, custom configuration blocks are typically placed inside the `/etc/httpd/conf.d/` drop-in directory (e.g., `/etc/httpd/conf.d/srv-web01.conf`).

> **Critical Architecture Trap:** In Apache, the `SSLUseStapling` mechanism requires an operational shared memory cyclic storage cache. The `SSLStaplingCache` directive **MUST be defined globally outside** of the `<VirtualHost>` isolation wrapper block.

```apache
# ==============================================================================
# GLOBAL SSL MODULAR CONFIGURATION (Define OUTSIDE VirtualHost blocks)
# ==============================================================================
SSLStaplingCache "shmcb:/run/httpd/ssl_stapling(32768)"

# ==============================================================================
# SECURE REVERSE PROXY / WEB SITE VIRTUAL HOST
# ==============================================================================
<VirtualHost *:443>
    ServerName srv-web01.local
    ServerAlias www.srv-web01.local
    DocumentRoot /var/www/html

    # ==========================================================================
    # CORE ENGINE HARDENING
    # ==========================================================================
    SSLEngine on
    
    # Modern Security Profile Enforcing TLS 1.2 and TLS 1.3
    SSLProtocol             all -SSLv3 -TLSv1 -TLSv1.1
    SSLHonorCipherOrder     off
    SSLCipherSuite          ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384

    # ==========================================================================
    # CERTIFICATE DIRECTIVES
    # ==========================================================================
    SSLCertificateFile      /etc/pki/tls/certs/srv-web01.local.crt
    SSLCertificateKeyFile   /etc/pki/tls/private/srv-web01.local.key
    
    # Injects the Intermediate CA to present a clear validation trace to clients
    SSLCertificateChainFile /etc/pki/tls/certs/intermediate.crt

    # ==========================================================================
    # REAL-TIME OCSP STAPLING ENGINE
    # ==========================================================================
    SSLUseStapling on
    
    # The Root CA file is required to verify the integrity of the OCSP signature locally
    SSLCACertificateFile    /etc/pki/tls/certs/ca.crt

    # Response validation constraints (Error buffer window limits)
    SSLStaplingReturnResponderErrors off
    SSLStaplingFakeTryLater off
    SSLStaplingResponseMaxAge 172800

    # HTTP Strict Transport Security Hardening (HSTS)
    Header always set Strict-Transport-Security "max-age=63072000"

    ErrorLog logs/srv-web01_error.log
    CustomLog logs/srv-web01_access.log combined
</VirtualHost>

```

