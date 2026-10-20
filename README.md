# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-20_13%3A17%3A16_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-20_13%3A17%3A16_UTC-green)
![Version](https://img.shields.io/badge/version-4d07bc627ecafd9d-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `4d07bc627ecafd9d7e7504f2ce55bf1e11bad5d842feff1dc38c6e83f75341f5` |
| **Last checked** | 2026-10-20 13:17:16 UTC |
| **Last updated** | 2026-10-20 13:17:16 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `5320c0b3b7ef11c9c6caf803b43700e4` |
| decrypt\_key | `621a20a28909879d4c22c94f22add6dd` |
| n\_key | `52828c908c877d29be3e6b7c725bffc8a7b2dbe841ba2fe7c901a2c505084de8` |
| fingerprint\_blob\_key | `6e0a383980add7291373206bb3adb116a4c1715ba424db7c6c94897d4a192b0b` |

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
| n\_key | `9a8954eba164197a06eeae729693dbef6461ea55ddf459a3cff0e6811444929d` |
| response\_decrypt\_key | `b4a35baac415108ee521be08377735cd7912c8016d2495739e6e92cf48f20518` |
| payload\_encrypt\_key | `7ded57d15a9e28388ed66fd193ed84368364d6852f598de6e46827567d518323` |

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
