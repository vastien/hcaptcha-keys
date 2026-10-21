# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-21_17%3A53%3A15_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-21_17%3A53%3A15_UTC-green)
![Version](https://img.shields.io/badge/version-a3e615664d7c033e-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `a3e615664d7c033ec779b0b9b519c4d047168e348dc9886937d1d3754dd80549` |
| **Last checked** | 2026-10-21 17:53:15 UTC |
| **Last updated** | 2026-10-21 17:53:15 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `32ed882ec2f4a893fea529d398d8e180` |
| decrypt\_key | `797e05068a9195494ee3bfa0bb409f6f` |
| n\_key | `dbc0e12904b8d8db7b12064c4ff5b7eb331e20a542967d3b9e35775ac55b88c4` |
| fingerprint\_blob\_key | `6e3be9bdf3e92dd3a9296f047dfdf32f98cf89b2a5605360f8e9070e28aa4315` |

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
| n\_key | `0c4ff0c623f9896c5409b3e8015d73d31960a3a8505578aa94b4c53fcbfb4ba9` |
| response\_decrypt\_key | `70f557ea2df11abde8c0fb5b0f8dcbc87834125458c29c89940854a4636a806e` |
| payload\_encrypt\_key | `d965679d228100dd542ddb7ac12fd482adcf0e86fa83d4b269f32b35eeecb593` |

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
