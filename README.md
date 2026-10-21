# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-21_12%3A51%3A33_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-21_12%3A51%3A33_UTC-green)
![Version](https://img.shields.io/badge/version-2e58092030771a60-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `2e58092030771a60ffcc547bb3f4c002a53f09877128d2f65b529786069891bc` |
| **Last checked** | 2026-10-21 12:51:33 UTC |
| **Last updated** | 2026-10-21 12:51:33 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `07288270b85346bd282823abc1d33938` |
| decrypt\_key | `c6a944b15ec38b9dee906bffc0628d60` |
| n\_key | `b0e31a583e9199ea6c5aaf3309f2ffdab9f47ce878f1351024a8a51e89e631fa` |
| fingerprint\_blob\_key | `038f69491c53e99f8e189f3ae3ccc196c4552d2ee41f7f330a9e35d96264c02e` |

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
| n\_key | `e2e21bf41004a5674e91febbf118f4c01bc5d3cca8f3f9bb7d094ac043bcdb69` |
| response\_decrypt\_key | `9b78c784eeb3c85b034b8ee486e46dcbda86cd2bae54793b530293344ba3b46b` |
| payload\_encrypt\_key | `27387469a01fc925e2849ab757f6b0f53436eb8488d2a57abffe29e897155a1f` |

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
