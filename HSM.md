# Fortanix DSM Integration with Chain-Fi Ecosystem

This document provides detailed instructions and architectural guidance for integrating **Fortanix DSM (Data Security Manager)** as a secure Hardware Security Module (HSM) within the Chain-Fi ecosystem, specifically tailored for backend systems hosted on DigitalOcean.

## Overview

Fortanix DSM offers cloud-native HSM services, providing secure key management, cryptographic signing, and encryption/decryption operations essential for regulatory compliance (GDPR, SOC 2, ISO 27001, FIPS 140-2).

## Why Fortanix DSM?

* **Regulatory Compliance:** FIPS 140-2 Level 3, GDPR, SOC 2 Type II, ISO 27001
* **Zero-Trust Security Model:** Keys never leave the Fortanix secure boundary
* **Comprehensive Audit Logging:** Detailed tracking for all cryptographic operations
* **Flexible and Scalable:** Supports multiple keys, key rotation, granular access control

## Architecture

```
[Chain-Fi Frontend / DApps]
          ↓
[DigitalOcean App Server]
          ↓ HTTPS with Mutual TLS
[Fortanix DSM HSM]
          ↓ (Performs Cryptographic Operations)
[Returns signed/encrypted result]
```

* **Zero Key Exposure:** Keys remain within Fortanix DSM, never on the app server.
* **Backend Signing Gateway:** DigitalOcean backend sends signing requests to Fortanix DSM.
* **Audit Logging:** All operations logged within Fortanix DSM for regulatory compliance.

## Integration Steps

### Step 1: Setup Fortanix DSM

1. Sign up at [Fortanix DSM](https://www.fortanix.com/).
2. Create a new application instance:

   * Define authentication method (Mutual TLS or API Key).
   * Configure key policies (sign-only, specific algorithms).

### Step 2: Generate and Manage Keys

Generate specific keys for different operations within Fortanix DSM:

| Key Label        | Usage                        | Algorithm   |
| ---------------- | ---------------------------- | ----------- |
| `vault-signer`   | Ethereum transaction signing | secp256k1   |
| `membership-key` | JWT/Membership validations   | Ed25519     |
| `stripe-hook`    | Stripe webhook verification  | HMAC-SHA256 |
| `admin-auth`     | Admin session tokens         | RSA-2048    |

### Step 3: Configure Mutual TLS (Recommended)

Securely connect your DigitalOcean-hosted backend to Fortanix DSM:

* Generate server and client certificates (mTLS).
* Configure Fortanix DSM to only accept requests from authorized servers via mTLS.

### Step 4: Backend Integration

Example API request for Ethereum transaction signing:

```javascript
const axios = require("axios");
const ethers = require("ethers");

const client = axios.create({
  baseURL: "https://<your-fortanix-instance>",
  httpsAgent: new https.Agent({
    cert: fs.readFileSync("./cert.pem"),
    key: fs.readFileSync("./key.pem"),
  }),
});

async function signTransaction(keyUuid, txData) {
  const hash = ethers.utils.keccak256(txData);
  const response = await client.post(`/crypto/v1/keys/${keyUuid}/sign`, {
    msg: Buffer.from(hash.slice(2), "hex").toString("base64"),
    alg: "ES256K",
    digest: true
  });
  return response.data.signature;
}
```

### Step 5: On-Chain Transaction Broadcasting

After signing, your backend receives the signature and broadcasts the transaction to Ethereum:

```javascript
const signedTx = ethers.utils.serializeTransaction(txData, signature);
const txResponse = await provider.sendTransaction(signedTx);
await txResponse.wait();
```

## Security Best Practices

* **Audit and Monitoring:** Regularly review Fortanix DSM audit logs.
* **Key Rotation:** Rotate keys periodically, maintaining versioning and backups within DSM.
* **Access Control:** Strictly manage user roles, limiting keys only to necessary applications.
* **IP Whitelisting and Geofencing:** Limit DSM access to your backend server IP range.

## Compliance and Audit

Fortanix DSM maintains compliance records and logs, simplifying SOC 2, ISO 27001, and GDPR audits:

* Detailed transaction logs
* Key usage and lifecycle reports
* Exportable logs for compliance documentation

## Support and Troubleshooting

* Fortanix DSM [documentation](https://support.fortanix.com/hc/en-us)
* Fortanix customer support: available directly via the Fortanix portal
* Chain-Fi development team for integration assistance

---

For further guidance, tailored integration scripts, or compliance support, contact the Chain-Fi development team directly.
