# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-18_22%3A12%3A43_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-18_22%3A12%3A43_UTC-green)
![Version](https://img.shields.io/badge/version-4e1f3b15d29c9d56-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `4e1f3b15d29c9d56ae6e0c3521dc21f6e33532294e6ed583875a7c5d9fa2306b` |
| **Last checked** | 2026-10-18 22:12:43 UTC |
| **Last updated** | 2026-10-18 22:12:43 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `b1000000a700000034000000de0000000000000030000000f3000000f3000000` |
| decrypt\_key | `0900000067000000df00000086000000ff00000030000000c30000000f000000` |
| n\_key | `f5f002bce822ccfa7a95d3c3af0d0d737c7d92e28fcfd4d2cb356c2d46dc79e5` |
| fingerprint\_blob\_key | `ae17a2f69fb3013fca3f225313932a14fa017d5df30a092bb2f4c5e981a65989` |

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
| n\_key | `d829f7bd011db21fd8f8f8194eeaedc3749fad9f7f03bee477f0a9180fead517` |
| response\_decrypt\_key | `1aa3e93f0f301977421f1633bc97be74a8fd375aa315f3a40b80d1f785c64fff` |
| payload\_encrypt\_key | `7724596431e026361562bae1c709795df22fe2c660c0b912ceae10a8367b6fd7` |

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
