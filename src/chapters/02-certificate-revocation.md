# Chapter 2: Certificate Revocation (CRL & OCSP)

To verify if an issued certificate is still trustworthy, two primary mechanisms are used across a Public Key Infrastructure: **Certificate Revocation Lists (CRLs)** and the **Online Certificate Status Protocol (OCSP)**.

While a certificate includes explicit validation dates, unforeseen security events—such as the compromise of a private key, corporate decommissioning, or data modifications—require the Certificate Authority (CA) to explicitly invalidate and revoke an active asset before its natural expiration date.

### Revocation Strategies Overview

1. **Certificate Revocation Lists (CRL)**: A signed, timestamped block containing blacklisted certificate serial numbers distributed to clients.
2. **Online Certificate Status Protocol (OCSP)**: A high-performance, real-time microservice interface that replies to status requests for individual certificates instantly without forcing clients to process heavy global lists.

This chapter details the exact mechanics of implementing both validation tracking schemas within an OpenSSL framework.

