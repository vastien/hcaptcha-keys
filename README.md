# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-19_12%3A21%3A00_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-19_12%3A21%3A00_UTC-green)
![Version](https://img.shields.io/badge/version-a099a810f5774cb6-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `a099a810f5774cb617085470978709e3fed07dafb9b030f6d32e31ed916d589c` |
| **Last checked** | 2026-10-19 12:21:00 UTC |
| **Last updated** | 2026-10-19 12:21:00 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `a83bae08c16b8ba35852bd39f8f42015` |
| decrypt\_key | `a58f325070ae0c774f73526908f4fa4e` |
| n\_key | `1f9806d420a5fc4c9372b31ea99e88895e8fbbf4aac183445a11ca58858a0b80` |
| fingerprint\_blob\_key | `1152939284cb4fd03ce358fedddcd4e32f6b78e42b593bfac38d2f433709d33b` |

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
| n\_key | `159c05dd95cb1c94398015101e210b62006de0986aea8c32435d2eb915c1fb30` |
| response\_decrypt\_key | `448a94c655ecaccd7deaf21fc0e897c27bd6cdbdaa7db6303ebfdfd3eefdf7c1` |
| payload\_encrypt\_key | `775a0496c83ec7d4cfcca45014c12960943b2adabbb97b791b7f6f2a775d793f` |

**HSJ wire format:** `ct(N) || tag(16) || iv(12) || 0x00`

## Programmatic Access

```javascript
const data = await fetch(
  'https://raw.githubusercontent.com/vastien/hcaptcha-keys/main/data/keys.json'
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
