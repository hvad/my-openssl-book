# Chapter 1: OpenSSL PKI Topology & Basics

For security reasons, a Root Certificate Authority (CA) is never used to sign 
end-entity server certificates directly. Instead, a strict hierarchical 
architecture is implemented to mitigate risks and isolate the primary trust anchor.

### Architectural Hierarchy

The standard practice utilizes a two-tier hierarchy:

```text
[ Root CA ]            <-- Kept offline, signs the Intermediate CA
     │
[ Intermediate CA ]    <-- Kept online, signs end-entity certificates
     │
[ Server Certificate ] <-- Used for Nginx, Apache, SSH, etc.

```

By following this layout, if an online system processing certificate requests is 
compromised, only the Intermediate CA is impacted. The primary Root CA remains secure 
and offline, allowing the operations team to issue a replacement intermediate tier without 
rebuilding the entire organizational trust foundation from scratch.

In this chapter, we will walk through the creation of this two-tier topology using manual 
OpenSSL primitives before moving into industrial automation patterns.

