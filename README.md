# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-20_01%3A05%3A17_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-20_01%3A05%3A17_UTC-green)
![Version](https://img.shields.io/badge/version-7267f9e30d127b96-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `7267f9e30d127b96bd1f737c55eb5bccf19d44571b59e6e972f0fc0fd17c427d` |
| **Last checked** | 2026-10-20 01:05:17 UTC |
| **Last updated** | 2026-10-20 01:05:17 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `7b315632f141eab51a8d8b42ebcf090a` |
| decrypt\_key | `8619a3f072b847af0ac4784294d4899b` |
| n\_key | `e6c3e7e3d62df6d2d98b4f3dfa3ee63964090102ff27e5d4f9e58f5eaa11c8bc` |
| fingerprint\_blob\_key | `7827029d3113682b09c3b09dd9fce0c75a23c3c6d6228575033a343cfe568e7d` |

**Key descriptions:**

- **encrypt\_key** — AES-256-GCM master key for encrypting outbound requests (amz magic via vc dispatcher). *(Requires WASM runtime patching — roadmap)*
- **decrypt\_key** — AES-256-GCM master key for decrypting server responses (tt magic via vc dispatcher). *(Requires WASM runtime patching — roadmap)*
- **n\_key** — AES-256 master key for the n-token cipher. Extracted from shadow memory at the fixslice AES entry (fn 330, reached via pc→389→250→548→fn330).
- **fingerprint\_blob\_key** — SHA-256(n\_key). Per-build deterministic identifier.

**Wire formats:**

~~~
HSW blob:  iv(12) || ct(N) || tag(16)
N-token:   AES-256-CTR, counter block = iv(12) || be32(counter)
~~~

## HSJ Keys

| Key Name | Value |
|----------|-------|
| n\_key | `603cc0099094119ae23226519e2ff73a6ac652853f58e2d0769f3f1ae8d8f096` |
| response\_decrypt\_key | `00c1d8fc43fc6513be53be4cc2f9d204176eb4f78c0f1fec87d85956025cd897` |
| payload\_encrypt\_key | `79a2a2d46c07a77e5c31e7bb263b9f0373aa7fe3c0fc953451b9d355c23e2833` |

**HSJ wire format:** `ct(N) || tag(16) || iv(12) || 0x00`

## Programmatic Access

```javascript
const data = await fetch(
  'https://raw.githubusercontent.com/vastien/hcaptcha-keys-tracker/main/data/keys.json'
).then(r => r.json());

const nKey            = data.hsw.n_key;             // 64 hex chars (32 bytes)
const fingerprintKey  = data.hsw.fingerprint_blob_key; // SHA-256 of n_key
const encryptKey      = data.hsw.encrypt_key;       // null until WASM patching is implemented
const decryptKey      = data.hsw.decrypt_key;       // null until WASM patching is implemented
```

## How Extraction Works

`n_key` and `fingerprint_blob_key` are extracted via **static binary scan** of the WASM data section:

1. Locate the active data segment with memory offset `0x14` and size `1004` bytes
2. K1 (16 bytes) = bytes at relative offset `0x15F` → mem[`0x173..0x182`]
3. K2 (16 bytes) = bytes at relative offset `0x16F` → mem[`0x183..0x192`]
4. `n_key` = K1 ‖ K2 (32 bytes concatenated)
5. `fingerprint_blob_key` = SHA-256(`n_key`)

These bytes live at the fixslice AES entry for the n-token cipher path and change with every new WASM build.

`encrypt_key` and `decrypt_key` require intercepting the fixslice32 key schedule
via WASM bytecode patching (finding the `vc` dispatcher, locating the key schedule
entry, injecting memory dumps). This is a planned enhancement.

## History

Previous build keys are archived in [data/history/](data/history/).

## License

MIT
