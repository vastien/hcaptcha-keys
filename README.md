# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-21_06%3A59%3A38_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-21_06%3A59%3A38_UTC-green)
![Version](https://img.shields.io/badge/version-c27ba394eff1c49e-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `c27ba394eff1c49e06a90bcda77eae7eeccc23e3b8fe6a2733397b848fafa0b6` |
| **Last checked** | 2026-10-21 06:59:38 UTC |
| **Last updated** | 2026-10-21 06:59:38 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `de65a0d8fb58ec308bca995aa4626c09` |
| decrypt\_key | `13133d7568a61013db526fcdd9e06f91` |
| n\_key | `d1720ec95d8c3ee85331a5467d4d0772e2012328060073e32f47e5f089818e04` |
| fingerprint\_blob\_key | `6196f601557d136aec6ce22c1d3756784df6bf83e579e4cc2893ba86eabb3042` |

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
| n\_key | `ab14b8ba07fac9ea53dd4dceb63395e70fc341d709d83d9a1a831e434db74771` |
| response\_decrypt\_key | `cf3e318fdc89aa852ab2eb1ecb7c59605e04efec72d369278b050925b18b646d` |
| payload\_encrypt\_key | `870a9df0ecae9c0e33bf729bdd0c223328b95e329c5552cc10de236a622aead2` |

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
