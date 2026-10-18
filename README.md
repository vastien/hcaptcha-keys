# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-18_19%3A12%3A58_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-18_19%3A12%3A58_UTC-green)
![Version](https://img.shields.io/badge/version-578c802eb448d959-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `578c802eb448d959e4d0ea0a00142397fa5230846abbf59d3af245ac01e5c1c0` |
| **Last checked** | 2026-10-18 19:12:58 UTC |
| **Last updated** | 2026-10-18 19:12:58 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `17000000ea000000a00000004c00000000000000f3000000f00000003c000000` |
| decrypt\_key | `460000001b000000400000007b0000000f000000c00000003f00000033000000` |
| n\_key | `540f94a16a2da38d6df424aa45b5e727de0602a758d4a841d6f1b8c953cbcb74` |
| fingerprint\_blob\_key | `134c156880843d5af98f6577d0440049771ab1b35e35c12b86db8611e5d816bb` |

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
| n\_key | `8287b3f8ac675a4f9e4a78b5e449b699db63fe62778e82b32d058835d5477215` |
| response\_decrypt\_key | `5b07d6af1ba8fed167d444cdaa1903a7265f655febc28a792f3345fdf4688e62` |
| payload\_encrypt\_key | `95ea4e868e8b40b9c9f4d227857442e756a2c65db3aec59d5e632df42281c950` |

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
