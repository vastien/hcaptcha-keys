# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-21_01%3A14%3A47_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-21_01%3A14%3A47_UTC-green)
![Version](https://img.shields.io/badge/version-64b7403d12a5db11-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `64b7403d12a5db11b435eb012bd0a1c92837a4462a87c031b0a83362bfbbb60a` |
| **Last checked** | 2026-10-21 01:14:47 UTC |
| **Last updated** | 2026-10-21 01:14:47 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `3508c7f60a59338d4978e54984a1923a` |
| decrypt\_key | `96080841db5c3227348ba1598f93ced9` |
| n\_key | `5abe011f8fc0dfd6377401affca7180ff367e29d841b36c45c48ecd2e797cf03` |
| fingerprint\_blob\_key | `4fcf401e4002b1fe84b464a28b6d2d5ec97032cca6bfff9c44aae461809c0b52` |

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
| n\_key | `01c3981c125ba35f0b0ccadb8b60268529dac24b81bd97d8fee149e822e755df` |
| response\_decrypt\_key | `b434e21ad98dafa8ded6266751ed66f5b53f80af407856e78ba8d5ee767df8a0` |
| payload\_encrypt\_key | `003cd036613097641e78ad44f1d21c69128b0326f9f018fbe918243760fdaaee` |

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
