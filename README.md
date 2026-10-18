# hCaptcha HSW/HSJ Key Tracker

> Automatically extracted cipher key material for every hCaptcha build. Updated hourly via GitHub Actions.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-18_16%3A36%3A12_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-18_16%3A33%3A18_UTC-green)
![Version](https://img.shields.io/badge/version-328395d3f1d70827-lightgrey)

## ⚠️ Disclaimer

This repository is for **educational and security research** purposes only. Use responsibly and in accordance with hCaptcha's terms of service.

## Current Build

| Field | Value |
|-------|-------|
| **Version hash** | `328395d3f1d70827e9bae48051926194029dd6ba330c7b9db7cdbc8387468d39` |
| **Last checked** | 2026-10-18 16:36:12 UTC |
| **Last updated** | 2026-10-18 16:33:18 UTC |

## HSW Cipher Keys

| Key | Value |
|-----|-------|
| encrypt\_key | `a0e7c66ca66624ce6fea75dd0a63e3bd` |
| decrypt\_key | `09c737323511a5a521805cd2b62b679a` |
| n\_key | `b0e4f2d6a6d8cd39037c6ec3bab4c2191a827286722c222eeca742444dbb92ad` |
| fingerprint\_blob\_key | `e80273cf20d3a94e29904578ccca1a1d9c22aa5d7dc761c825b7605e38f9f08c` |

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
| n\_key | *(not extracted)* |
| response\_decrypt\_key | *(not extracted)* |
| payload\_encrypt\_key | *(not extracted)* |


## Programmatic Access

```javascript
const data = await fetch(
  'https://raw.githubusercontent.com/YOUR_USER/hcaptcha-keys-tracker/main/data/keys.json'
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
