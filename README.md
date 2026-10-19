# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-19_05%3A40%3A52_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-19_05%3A40%3A52_UTC-green)
![Version](https://img.shields.io/badge/version-8f868d9715739742-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `8f868d971573974267b492b11ab18b2e09e8ff8f11e565737c7b66565db23f8f` |
| **Last checked** | 2026-10-19 05:40:52 UTC |
| **Last updated** | 2026-10-19 05:40:52 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `60000000d50000009a0000009d0000000000000030000000f0000000c0000000` |
| decrypt\_key | `9a00000050000000ac000000a6000000fc00000030000000c00000003f000000` |
| n\_key | `c75a37ffa69e3192c337e3e4395bd613ffeb6da1e5edbb7b8deb913e8492a3c9` |
| fingerprint\_blob\_key | `f3c0c7f63921d5f62a11dcac53515168eea545e8f73df2ffd2cb129fdd9b06ee` |

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
| n\_key | `694523ec911c60bd5f03fb4accbd999220a3650f5df8099762c3aedcb4ac69ae` |
| response\_decrypt\_key | `9f82da225fd1fba719d159afb727e148c30a9ee2e998750b68bb8021ed6f5692` |
| payload\_encrypt\_key | `c4b2902f4e31ff13aaa41cf0a086e0c9d726b935b48552caee334d7d183cb24d` |

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
