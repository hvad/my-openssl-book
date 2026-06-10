# 4.2 Nginx Server Block Configuration

Create or update your virtual host file layout definition blueprint (e.g., `/etc/nginx/sites-available/srv-web01.conf`). The following deployment block is optimized for **TLS 1.3**, enforces HTTP-to-HTTPS redirection, and embeds the background verification routines required for OCSP Stapling:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name srv-web01.local www.srv-web01.local;
    
    # Enforce global automatic HTTP to HTTPS redirection
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name srv-web01.local www.srv-web01.local;

    root /var/www/html;
    index index.html;

    # ==============================================================================
    # CERTIFICATE PATH CONFIGURATION
    # ==============================================================================
    ssl_certificate         /etc/nginx/ssl/srv-web01.fullchain.pem;
    ssl_certificate_key     /etc/nginx/ssl/srv-web01.local.key.pem;

    # ==============================================================================
    # HARDENED TLS PARAMETERS (Modern Security Profile)
    # ==============================================================================
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384';

    # Performance and Handshake Session Optimization
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;

    # ==============================================================================
    # AUTOMATED OCSP STAPLING ENGINE
    # ==============================================================================
    ssl_stapling on;
    ssl_stapling_verify on;

    # Nginx must parse the Root CA to cryptographically verify the OCSP responder signature
    ssl_trusted_certificate /etc/nginx/ssl/ca.cert.pem;

    # DNS Resolver: Internal engine coordinate to resolve the embedded OCSP URI
    # (Replace with your local corporate DNS gateway or a reliable public provider)
    resolver 192.168.1.1 8.8.8.8 valid=300s;
    resolver_timeout 5s;

    # HTTP Strict Transport Security (HSTS Hardening)
    add_header Strict-Transport-Security "max-age=63072000" always;

    location / {
        try_files $uri $uri/ =404;
    }
}

```

