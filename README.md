# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-20_22%3A03%3A51_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-20_22%3A03%3A51_UTC-green)
![Version](https://img.shields.io/badge/version-687003db6c78cd15-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `687003db6c78cd156e5a8810ba646d2eb14e70eb6ee9869e80a116bea616fe31` |
| **Last checked** | 2026-10-20 22:03:51 UTC |
| **Last updated** | 2026-10-20 22:03:51 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `16f7e9f3fd9f22f52d51b0167b42eee6` |
| decrypt\_key | `b4c24e804fde2f2e4c0d72876acf5067` |
| n\_key | `343281c0a2fca67d4d045a8b209985dbdc3b539aca48bdde9fee8b0cb3d6cd63` |
| fingerprint\_blob\_key | `cd14bcf87e5a4ac6958434d514fa743d324d5abedcaaf452130e7221167b95da` |

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
| n\_key | `e7fb03f2051ea993e82898802c584dfd0a774ba70315075b7588b9b69bb422b6` |
| response\_decrypt\_key | `29e2439847c135d7c7d5d0b154bfac6686f1b9c22334b488f4e9ae9546285daa` |
| payload\_encrypt\_key | `2c4c12a645bab33837fdf5effec1fd893ccad6a169e41e1aef6dce5a10273e83` |

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
