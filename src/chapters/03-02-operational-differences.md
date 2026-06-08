# 3.2 Key Operational Differences

Automating infrastructure tasks changes how the underlying cryptography engine responds. 
Below are the key engineering advancements implemented in the automation script compared to manual setup workflows:

### Non-Interactive Execution (`-batch`)
The `openssl ca` signing command includes the `-batch` flag wrapper attribute argument. 
By default, manual execution pauses operations to ask confirmation questions:
* *Sign the certificate? [y/n]*
* *1 out of 1 certificate requests certified, commit database [y/n]?*

The `-batch` attribute suppresses interactive confirmation interfaces entirely. 
It allows the toolchain to run completely unattended inside continuous delivery pipelines or cron-scheduled maintenance loops.

### Runtime Configuration Insertion (`-passin pass:xxx`)
Instead of prompting an engineer to enter a security passphrase manually at the terminal prompt, 
the script maps parameters via `-passin pass:"$INT_PASS"`. This safely handles cryptographic challenges 
within volatile configuration context arrays, eliminating shell hangs.

### Dynamic Extension Compilation (`-addext`)
During the leaf certificate creation phase, the script uses the `-addext` parameter configuration 
definition block to inject **Subject Alternative Names (SAN)** inside the raw Certificate Signing Request dynamically:

```bash
-addext "subjectAltName = DNS:$SERVER_NAME, DNS:www.$SERVER_NAME, IP:$IP_SERVER"

```

This bypasses the need to create, parse, track, and clean up temporary static local `.cnf` files 
on the host filesystem before triggering compilation pipelines.

