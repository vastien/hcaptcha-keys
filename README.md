# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-19_00%3A45%3A46_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-19_00%3A45%3A46_UTC-green)
![Version](https://img.shields.io/badge/version-a89a2651a82210f3-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `a89a2651a82210f3a2af6721ccffb5c6d37b803c394a040808e58df85e9c9110` |
| **Last checked** | 2026-10-19 00:45:46 UTC |
| **Last updated** | 2026-10-19 00:45:46 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `7b00000003000000340000006900000000000000fc000000c3000000cf000000` |
| decrypt\_key | `970000004e0000008d00000058000000ff000000cc00000030000000cf000000` |
| n\_key | `9ded86dbd1719dbb570ded3a1a2839df602df7147afa9655a97028a7c06140e7` |
| fingerprint\_blob\_key | `e6b51c3e51fdc85c25541d91a80f7064d353964ec5c11329b44b57c21c6f0e8e` |

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
| n\_key | `24051fbaa7ac82dd290676bf216c8054537b1e84030cc275e1d211a3ac0667b6` |
| response\_decrypt\_key | `90124046370fce5ee9dce5b5c62c10de4512e7691212bfa79e6292dfc3425596` |
| payload\_encrypt\_key | `bf5553c8c009ac4ed05d50753985cd8e3fb9cbb8056f8aa810631a094393f602` |

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
