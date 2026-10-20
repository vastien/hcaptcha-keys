# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-20_18%3A18%3A34_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-20_18%3A18%3A34_UTC-green)
![Version](https://img.shields.io/badge/version-272aac4d659aaee8-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `272aac4d659aaee81259ce9b8d2946e9a2cd79ba9a7f04e699b9e2be4f2ac775` |
| **Last checked** | 2026-10-20 18:18:34 UTC |
| **Last updated** | 2026-10-20 18:18:34 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `b0c40fe711e2d197622722489d6f80d4` |
| decrypt\_key | `3ae6ca444bfdd9a06426234c4a73d182` |
| n\_key | `c53f9d012cdb07d3a0de816a5d7df6cace03d906a5e45a9bd2246ceed8cf6151` |
| fingerprint\_blob\_key | `f37006817eebd8ea9afc30f8e25c6b0642a3e5cbe02ddcb3967b41fd152484b5` |

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
| n\_key | `48dc8d912ce037f2b026d65e060c1a197a2b0a093fdca1e02941d0b957497031` |
| response\_decrypt\_key | `baaa5374645e4db320a6dec804d8aff311f9673eae1a77d327b9db6d92bdc299` |
| payload\_encrypt\_key | `0da390b0c879e97bcd6d2d590b5cd2250c2be5ec1b93a72e996d0db537baa350` |

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
