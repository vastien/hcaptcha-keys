# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-21_21%3A12%3A43_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-21_21%3A12%3A43_UTC-green)
![Version](https://img.shields.io/badge/version-fb0aa9d1ee9091dc-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `fb0aa9d1ee9091dc2637ff9cb4215f20259b57a0b0e48c190473a1d3b2a45c71` |
| **Last checked** | 2026-10-21 21:12:43 UTC |
| **Last updated** | 2026-10-21 21:12:43 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `ad1fd7b222c342465aaed096a0ebb4a0` |
| decrypt\_key | `f2d63a8e27fd6937c1130df97bf398d5` |
| n\_key | `739186c9f4f63d0cbcdaf68b5f2fc1d95c2c972f29b245e3554ebc9376184727` |
| fingerprint\_blob\_key | `6c59c6ae832f42625d91e30aa727cff3c3ff286652db38e1b19f41998444f6e7` |

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
| n\_key | `9d1fc0e2ff776aaaeb948e99dc3159faf7a54fb1ef4ec36b6a3230b23c7da92b` |
| response\_decrypt\_key | `cf4184d52a2b0040ba248aab4225fcd6fa763f560318d6c02fe134ea76d177bd` |
| payload\_encrypt\_key | `be033a2ea6536abd06dfe8c47e8585989208aec13072d906548f5f648ea0aad3` |

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
