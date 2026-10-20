# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-20_01%3A32%3A05_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-20_01%3A32%3A05_UTC-green)
![Version](https://img.shields.io/badge/version-f9b1f2d6aab0a8f3-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `f9b1f2d6aab0a8f3c180b9475fe2f594c36dc9aa98928fcbd20e19cc3c141859` |
| **Last checked** | 2026-10-20 01:32:05 UTC |
| **Last updated** | 2026-10-20 01:32:05 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `e2799f983377206ab35dc762c7da71de` |
| decrypt\_key | `4a702b4385ad4443075b7ae9edb5547e` |
| n\_key | `e6dd022acee62a8c81af533b782a612a56c0e65a7b2d120351847b026b9a4f37` |
| fingerprint\_blob\_key | `f62a835d65533baaaa5acda723baa72e4e3264bfad19cd64f52490da700c10b7` |

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
| n\_key | `0f2eb0b7ce65e8c4a9a44716ca17538ef83e072cdaecf0b1309687f31bcb4e34` |
| response\_decrypt\_key | `fa4165679292619ce773f0da32f36574e8ee0cc7216a716fd4a6f0ef37bba84d` |
| payload\_encrypt\_key | `b3a4fc1819ed325c6dd4a73cefbd1d0aa3525828ceb3adbfffaaf8f1672a6d4d` |

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
