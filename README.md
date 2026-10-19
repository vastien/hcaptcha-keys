# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-19_18%3A40%3A53_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-19_18%3A40%3A53_UTC-green)
![Version](https://img.shields.io/badge/version-cb9dca9bdb6bb065-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `cb9dca9bdb6bb0650a81de3fda48c6eb0e5f2f2787e3b7564182f19601f643d5` |
| **Last checked** | 2026-10-19 18:40:53 UTC |
| **Last updated** | 2026-10-19 18:40:53 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `f47095d21cef4baa8f208145998bc84b` |
| decrypt\_key | `560332241b78cf7f4120bfc1b59cb9c3` |
| n\_key | `0289563b3309b018efdb8acfd4224ef6c618f6265e26aa9f7a5410dbd4a85691` |
| fingerprint\_blob\_key | `cbbadb70cd2f6f8ba628cbae8e5e1b9fc699c9627d0d61bd2b0dfb156b084a5a` |

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
| n\_key | `cfc516b9c93bf8be37cd6ce9708e802ea80b6c91033f16f8ac482e1d688724f3` |
| response\_decrypt\_key | `ae76e998bfa1cb46fb48a3c9e8d04409b55a8776cab5fb4af44e99fc5092fdbe` |
| payload\_encrypt\_key | `4117eaf62e2b60dd484ef935bb489e98c6c5e2d72142189ec8d588e3e61d3e50` |

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
