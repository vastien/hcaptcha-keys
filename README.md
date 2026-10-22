# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-22_00%3A25%3A14_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-22_00%3A25%3A14_UTC-green)
![Version](https://img.shields.io/badge/version-185618a1b4eaeca4-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `185618a1b4eaeca4921e61756ba5d7d85ffec4787df799c57c2f6e7accec0fcb` |
| **Last checked** | 2026-10-22 00:25:14 UTC |
| **Last updated** | 2026-10-22 00:25:14 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `91b99b8d205bd4563b38737247edd0e2` |
| decrypt\_key | `e2eccdf4546ea9eda60e857deb493ec5` |
| n\_key | `7197a2cad968fab474f57fb0de1f22ae5e4f83fff8a72fe0112f76c6a32c8068` |
| fingerprint\_blob\_key | `e02237a0c76fb722bbf908346fc49b4bb2f0caff735b95dfddd5f9671a07842f` |

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
| n\_key | `1c43f88c17a7c734429cf22c520e6e241c6412489b51594c178dae40597d2543` |
| response\_decrypt\_key | `6d0750a821dc1cd2fee5e7b8caedf77fbb5773b1b590a144a89cbd948078cb4b` |
| payload\_encrypt\_key | `b1c33b118fe8bade4f7de10610a045384d1db74b07a0b38faeb79eb35cf62d7e` |

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
