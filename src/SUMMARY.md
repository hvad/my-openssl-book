# Summary

[Introduction](README.md)

- [Chapter 1: OpenSSL PKI Topology & Basics](chapters/01-pki-topology-basics.md)
    - [1.1 Root CA Creation](chapters/01-01-root-ca-creation.md)
    - [1.2 Intermediate CA Creation](chapters/01-02-intermediate-ca-creation.md)
    - [1.3 Server Certificate Generation](chapters/01-03-server-certificate-generation.md)
    - [1.4 Useful Verification Commands](chapters/01-04-verification-commands.md)
    - [1.5 Deployment & Full Chain Files](chapters/01-05-full-chain-deployment.md)

- [Chapter 2: Certificate Revocation (CRL & OCSP)](chapters/02-certificate-revocation.md)
    - [2.1 Certificate Revocation Lists (CRL)](chapters/02-01-crl-configuration.md)
    - [2.2 Online Certificate Status Protocol (OCSP)](chapters/02-02-ocsp-responder.md)
    - [2.3 Testing & Verifying Revocation](chapters/02-03-testing-revocation.md)
    - [2.4 Introduction to OCSP Stapling](chapters/02-04-intro-ocsp-stapling.md)

- [Chapter 3: PKI Automation with Bash](chapters/03-pki-automation-bash.md)
    - [3.1 The Deployment Script (deploy-pki.sh)](chapters/03-01-automation-script.md)
    - [3.2 Key Operational Differences](chapters/03-02-operational-differences.md)

- [Chapter 4: Web Server Configuration - Nginx OCSP Stapling](chapters/04-nginx-ocsp-stapling.md)
    - [4.1 Prerequisites & File Deployment](chapters/04-01-nginx-prerequisites.md)
    - [4.2 Nginx Server Block Configuration](chapters/04-02-nginx-configuration.md)
    - [4.3 Testing and Verifying Nginx Stapling](chapters/04-03-nginx-verification.md)

- [Chapter 5: Web Server Configuration - Apache HTTPD OCSP Stapling](chapters/05-apache-ocsp-stapling.md)
    - [5.1 File Placement Standards](chapters/05-01-apache-file-placement.md)
    - [5.2 Apache VirtualHost Configuration](chapters/05-02-apache-configuration.md)
    - [5.3 Validation & Troubleshooting](chapters/05-03-apache-validation.md)
