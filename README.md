# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-20_07%3A06%3A28_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-20_07%3A06%3A28_UTC-green)
![Version](https://img.shields.io/badge/version-bdc9f497d606caf3-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `bdc9f497d606caf396e4b898b63bfb8e59a1cbd49ec6ec28a62a819845881921` |
| **Last checked** | 2026-10-20 07:06:28 UTC |
| **Last updated** | 2026-10-20 07:06:28 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `54c9bba756e34b142ef911b69a632eca` |
| decrypt\_key | `806e5628401ddc2b90f2afb608702969` |
| n\_key | `cb5838c1950ed684ee4e8ce82333819a168df921cc14c3bffcee32d193357f25` |
| fingerprint\_blob\_key | `e0abb0700d68bf7a7114c381917f5a167ebf3bab8f0c417fed2f5a0c0383b527` |

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
| n\_key | `e1c233ddbe5bffb8b3545118219a5505e477ee4940af1fa00e31a1073241897d` |
| response\_decrypt\_key | `2bc4741b1e5e0c19ade4bfbb9c139e326760a7fa35afbbed5568ab24615a1eec` |
| payload\_encrypt\_key | `e51fa5d9f43b00e52363e9476d615ad9df7c4414d14615b3be8d1c47bde73e51` |

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
