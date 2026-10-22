# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-22_11%3A05%3A42_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-22_11%3A05%3A42_UTC-green)
![Version](https://img.shields.io/badge/version-8baa4b835f807ff1-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `8baa4b835f807ff11777c7bff3bdce73acc263c771f670a7bbfb92af04611861` |
| **Last checked** | 2026-10-22 11:05:42 UTC |
| **Last updated** | 2026-10-22 11:05:42 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `ab85dde626618278ea0d459643ea9dfe` |
| decrypt\_key | `7933e5b31e47b6290ac2153aafaf4e68` |
| n\_key | `683bf4c1e72eff7b80d039afbdd8082a97396be3350284b68add7997708f8e5a` |
| fingerprint\_blob\_key | `1a4875b3c42befb3c300b7c241be560f35fb6b95821493890efde54e8dfea398` |

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
| n\_key | `88c931dd1847497532de441eafc4d98e00e4ad2fefd77343ba8ce8f7812f0dca` |
| response\_decrypt\_key | `0da0e249a76eb3c7942a8e39af215a29243be365354f6d9bf7abd3e967675349` |
| payload\_encrypt\_key | `81188b4150d85576120aadc22ee983ecd5070be375d5379f22774a5c7fbd10c1` |

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
