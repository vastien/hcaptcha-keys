# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-19_22%3A48%3A14_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-19_22%3A48%3A14_UTC-green)
![Version](https://img.shields.io/badge/version-85dd63f6a4e8742e-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `85dd63f6a4e8742e0be4dd839d30a6424ed4f998f85d7c8aefd5fe7f4263f0c2` |
| **Last checked** | 2026-10-19 22:48:14 UTC |
| **Last updated** | 2026-10-19 22:48:14 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `634f70ed380373ca5cd9d19b01c56778` |
| decrypt\_key | `3bec9d5659e7c0a90bc8b673870651c3` |
| n\_key | `1f9de8fc9a010c228165911ffef3d33a97b20a2e837f1b7363b8156c953e64ae` |
| fingerprint\_blob\_key | `790008b6fa189aabc00e3d6b67f36defec8cc8b4f8ce1a70819df62f1e040c12` |

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
| n\_key | `22be674a119562598ebe553f49a27889613610fbc331f81ea076dfe3db373ce7` |
| response\_decrypt\_key | `2a33d7d830c6807fc8e0baedb3e12f32b22f193f95f0ce051fee7b3a069df119` |
| payload\_encrypt\_key | `b1ac1eeb114707a7aef94a06e255dd6451a51f3946f48110ef0c8c73ef9aa855` |

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
