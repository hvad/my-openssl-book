# 3.1 The Deployment Script (deploy-pki.sh)

Create a file named `deploy-pki.sh`, copy the automation engine codebase below into it, and make it executable (`chmod +x deploy-pki.sh`).

```bash
#!/bin/bash

# ==============================================================================
# CONFIGURATION AND VARIABLES
# ==============================================================================
set -e # Terminate execution instantly if any command fails

BASE_DIR="$HOME/pki-auto"
ROOT_DIR="$BASE_DIR/root-ca"
INT_DIR="$BASE_DIR/intermediate"

# Default Operational Passphrases (CRITICAL: Change these for production!)
ROOT_PASS="SuperSecureRootPass123!"
INT_PASS="SuperSecureIntPass123!"

# Distinguished Name (DN) Identity Metadata Configuration Parameters
COUNTRY="FR"
ORG="My Company"
ROOT_CN="My Root PKI Authority"
INT_CN="My Intermediate PKI Authority"

echo "=============================================================================="
echo " Deploying Automated OpenSSL PKI Infrastructure"
echo "=============================================================================="

# ==============================================================================
# 1. DIRECTORY STRUCTURE INITIALIZATION
# ==============================================================================
echo "[*] Initializing filesystem layout structure..."
mkdir -p "$ROOT_DIR"/{certs,crl,newcerts,private}
mkdir -p "$INT_DIR"/{certs,crl,csr,newcerts,private}

chmod 700 "$ROOT_DIR/private" "$INT_DIR/private"

# Initialize OpenSSL flat-file inventory database tracking arrays
touch "$ROOT_DIR/index.txt" "$INT_DIR/index.txt"
echo 1000 > "$ROOT_DIR/serial"
echo 1000 > "$INT_DIR/serial"

# ==============================================================================
# 2. OPENSSL BLUEPRINT CONFIGURATION GENERATION
# ==============================================================================
echo "[*] Injecting customized openssl.cnf profile configurations..."

cat <<EOF > "$INT_DIR/openssl.cnf"
[ ca ]
default_ca = CA_default

[ CA_default ]
dir               = $INT_DIR
certs             = \$dir/certs
crl_dir           = \$dir/crl
new_certs_dir     = \$dir/newcerts
database          = \$dir/index.txt
serial            = \$dir/serial
private_key       = \$dir/private/intermediate.key.pem
certificate       = \$dir/certs/intermediate.cert.pem

crlnumber         = \$dir/crlnumber
crl               = \$dir/crl/intermediate.crl

default_md        = sha256
name_opt          = ca_default
cert_opt          = ca_default
default_days      = 365
preserve          = no
policy            = policy_loose

[ policy_loose ]
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ server_cert ]
subjectKeyIdentifier    = hash
authorityKeyIdentifier  = keyid,issuer
basicConstraints        = CA:FALSE
keyUsage                = critical, digitalSignature, keyEncipherment
extendedKeyUsage        = serverAuth
crlDistributionPoints   = URI:[http://pki.monentreprise.local/crl/intermediate.crl](http://pki.monentreprise.local/crl/intermediate.crl)
authorityInfoAccess     = OCSP;URI:[http://ocsp.monentreprise.local:2560](http://ocsp.monentreprise.local:2560)
EOF

# ==============================================================================
# 3. ROOT CA INITIALIZATION FOUNDATION
# ==============================================================================
if [ ! -f "$ROOT_DIR/certs/ca.cert.pem" ]; then
    echo "[*] Generating Root CA Trust Anchor..."
    
    # Generate encrypted high-entropy Root Private Key
    openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 \
        -aes256 -pass pass:"$ROOT_PASS" \
        -out "$ROOT_DIR/private/ca.key.pem"
        
    # Generate Self-Signed Root Anchor Certificate
    openssl req -new -x509 -days 3650 -sha256 \
        -passin pass:"$ROOT_PASS" \
        -key "$ROOT_DIR/private/ca.key.pem" \
        -subj "/C=$COUNTRY/O=$ORG/CN=$ROOT_CN" \
        -out "$ROOT_DIR/certs/ca.cert.pem"
else
    echo "[+] Root CA Trust Anchor already exists. Skipping initialization."
fi

# ==============================================================================
# 4. INTERMEDIATE CA SIGNING LAYER INITIALIZATION
# ==============================================================================
if [ ! -f "$INT_DIR/certs/intermediate.cert.pem" ]; then
    echo "[*] Generating Intermediate Signing CA Layer..."
    
    # Generate encrypted Intermediate Private Key
    openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 \
        -aes256 -pass pass:"$INT_PASS" \
        -out "$INT_DIR/private/intermediate.key.pem"
        
    # Create the Intermediate CSR
    openssl req -new -sha256 \
        -passin pass:"$INT_PASS" \
        -key "$INT_DIR/private/intermediate.key.pem" \
        -subj "/C=$COUNTRY/O=$ORG/CN=$INT_CN" \
        -out "$INT_DIR/csr/intermediate.csr.pem"
        
    # Standard temporary inline configuration override block for Root CA execution
    cat <<EOF > "$ROOT_DIR/openssl_root.cnf"
[ ca ]
default_ca = CA_root
[ CA_root ]
dir               = $ROOT_DIR
database          = \$dir/index.txt
serial            = \$dir/serial
private_key       = \$dir/private/ca.key.pem
certificate       = \$dir/certs/ca.cert.pem
new_certs_dir     = \$dir/newcerts
default_md        = sha256
policy            = policy_any
[ policy_any ]
domainComponent         = optional
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
[ v3_intermediate_ca ]
subjectKeyIdentifier    = hash
authorityKeyIdentifier  = keyid:always,issuer
basicConstraints        = critical, CA:true, pathlen:0
keyUsage                = critical, digitalSignature, cRLSign, keyCertSign
EOF

    # Sign the Intermediate layer using the Root CA Trust Anchor
    openssl ca -config "$ROOT_DIR/openssl_root.cnf" \
        -extensions v3_intermediate_ca -days 3650 -notext -md sha256 -batch \
        -passin pass:"$ROOT_PASS" \
        -in "$INT_DIR/csr/intermediate.csr.pem" \
        -out "$INT_DIR/certs/intermediate.cert.pem"
        
    rm -f "$ROOT_DIR/openssl_root.cnf"
else
    echo "[+] Intermediate CA Layer already exists. Skipping initialization."
fi

# ==============================================================================
# 5. DYNAMIC TARGET SERVER CERTIFICATE ISSUANCE PROCESS
# ==============================================================================
SERVER_NAME="srv-web01.local"
IP_SERVER="192.168.1.100"

echo "[*] Provisioning Automated Leaf Certificate Asset for: $SERVER_NAME"

# 1. Generate an unencrypted target Server Private Key (Optimized for headless restarts)
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 \
    -out "$INT_DIR/private/$SERVER_NAME.key.pem"

# 2. Create the target Server CSR embedding Subject Alternative Names (SAN) on the fly
openssl req -new -sha256 \
    -key "$INT_DIR/private/$SERVER_NAME.key.pem" \
    -subj "/C=$COUNTRY/O=$ORG/CN=$SERVER_NAME" \
    -addext "subjectAltName = DNS:$SERVER_NAME, DNS:www.$SERVER_NAME, IP:$IP_SERVER" \
    -out "$INT_DIR/csr/$SERVER_NAME.csr.pem"

# 3. Process signing execution against Intermediate CA engine profile configurations
openssl ca -config "$INT_DIR/openssl.cnf" \
    -extensions server_cert -days 365 -notext -md sha256 -batch \
    -passin pass:"$INT_PASS" \
    -in "$INT_DIR/csr/$SERVER_NAME.csr.pem" \
    -out "$INT_DIR/certs/$SERVER_NAME.cert.pem"

# 4. Generate the finalized aggregate Full Chain compilation file
cat "$INT_DIR/certs/$SERVER_NAME.cert.pem" "$INT_DIR/certs/intermediate.cert.pem" > "$INT_DIR/certs/$SERVER_NAME.fullchain.pem"

echo "=============================================================================="
echo "[+] Success! Compiled Full Chain available at:"
echo "    $INT_DIR/certs/$SERVER_NAME.fullchain.pem"
echo "=============================================================================="

```

