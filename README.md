# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-22_05%3A43%3A56_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-22_05%3A43%3A56_UTC-green)
![Version](https://img.shields.io/badge/version-d3f70ab8f47a9643-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `d3f70ab8f47a9643cc64c34f0eccc9e601f10eb04939f0d4562bfa92b8977cff` |
| **Last checked** | 2026-10-22 05:43:56 UTC |
| **Last updated** | 2026-10-22 05:43:56 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `26074bcf4c8b23d9af61cabdf864f18d` |
| decrypt\_key | `6a0f1b4b18a156160a5302fbd4290164` |
| n\_key | `f22358815b79ed45eaa3eb00d0b0b8960bed531ab9e894901b1c6f6c65c8ee0f` |
| fingerprint\_blob\_key | `8ec0e806b973f84fe2328549781bfffa6262f8e6e48fcb314710605c38e42418` |

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
| n\_key | `9c861aa3317df8fd2652a7067416175c42e0e62ef7d8a4a8254a367c5ef8486f` |
| response\_decrypt\_key | `f763b8486d1dd89b17e7fa69c41211fb3efb9a0c44a36f9baacba1c1b0a27a84` |
| payload\_encrypt\_key | `1c3b0caba9fa0c334711523d925b6cdf3b111cf9d960b02dbd523d2950813ba7` |

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
