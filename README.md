# hCaptcha HSW Key Tracker

> The two AES-256-GCM keys of each hCaptcha `hsw.js` build, extracted from the
> build's own WASM module and confirmed against it before being published.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-22_23%3A52%3A22_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-22_23%3A52%3A22_UTC-green)
![Build](https://img.shields.io/badge/build-315a0135ff767aff-lightgrey)
![Keys confirmed](https://img.shields.io/badge/keys_confirmed-both-brightgreen)

## ⚠️ Disclaimer

For **educational and security research** purposes only. Use responsibly and in
accordance with hCaptcha's terms of service.

## Keys are per build, and builds rotate several times a day

A key is only valid for the `hsw.js` build it came from. hCaptcha ships several
builds a day, and nothing survives a rotation — not the keys, and not a single
WASM function index. So what this repository offers is a **re-runnable
extractor** (about 5 seconds per build), not a constant.

To know which build a capture used, read the `/getcaptcha` **request** body: it
carries the proof-spec JWT, whose `l` claim is the bundle path and whose `i`
claim is that bundle's SRI hash. Keys here are indexed by that build hash.

## Current build

| Field | Value |
|-------|-------|
| **Build** | `315a0135ff767aff1a14afa37e2aed714289cf7479391fa459097230b428d931` |
| **Bundle** | `https://newassets.hcaptcha.com/c/315a0135ff767aff1a14afa37e2aed714289cf7479391fa459097230b428d931/hsw.js` |
| **Bundle SRI** | `sha256-QLBIEr+5U2Ba3OOT2eQOxB/AXr5S6L7aUJB6NTEkEiM=` |
| **Runtime WASM sha256** | `02919f832a1f0819112d34da9423c31fcfc5b4914e5bc687fcdd4e9793d92c9a` |
| **Last checked** | 2026-10-22 23:52:22 UTC |
| **Last updated** | 2026-10-22 23:52:22 UTC |

## HSW keys

Cipher: **AES-256-GCM**. Wire format both ways:
`iv(12) || ciphertext(N) || tag(16)`.

| Key | Value | What it is for |
|-----|-------|----------------|
| request\_key | `84dff2c4ab5b6c657a7ecfb6b0436243972729c346265f131b2a7d9822fd07c8` | Encrypts the outbound request body — the key `hsw(1, plaintext)` uses |
| response\_key | `a4c213bc97c2fa8c59b7d96f453762115d8b45d680cad0408b12abe7ade9af9b` | Decrypts the `/getcaptcha` response — the key `hsw(0, blob)` uses |

They are **two different keys**, and neither one works in place of the other.

### How each one is confirmed

Nothing is published unless the build's own module agrees with it:

~~~
request_key  K  is accepted iff  GCM-open(K, hsw(1, pt))          == pt
response_key K  is accepted iff  hsw(0, iv || GCM(K,iv,pt) || tag) == pt
~~~

A candidate that fails is discarded, and a build where neither key can be
confirmed fails the run and leaves the previous data untouched.

## The n-token has no key

Earlier versions of this repository published an `n_key` and a
`fingerprint_blob_key`, for hsw and for hsj. All of them have been removed:
they were wrong. The n-token is
**not** encrypted under a recoverable key at all — it is a deterministic
pipeline (a 31-bit repack, five modular passes and a word rotation, XORed
against a fixed keystream with no key, nonce or per-call entropy). There is no
n-token key to publish, so this repository no longer pretends there is one.

## Programmatic access

```javascript
const data = await fetch(
  'https://raw.githubusercontent.com/vastien/hcaptcha-keys/main/data/keys.json'
).then(r => r.json());

// Check the build matches your capture before using these.
if (data.version !== myCaptureBuildHash) throw new Error('key is for a different build');

const requestKey  = Buffer.from(data.hsw.request_key, 'hex');   // 32 bytes
const responseKey = Buffer.from(data.hsw.response_key, 'hex');  // 32 bytes
```

```javascript
// decrypt a /getcaptcha response: iv(12) || ct || tag(16)
const iv = blob.subarray(0, 12);
const tag = blob.subarray(blob.length - 16);
const ct = blob.subarray(12, blob.length - 16);
const d = crypto.createDecipheriv('aes-256-gcm', responseKey, iv);
d.setAuthTag(tag);
const plaintext = Buffer.concat([d.update(ct), d.final()]);  // msgpack, not JSON
```

## This build's function mapping

Recorded for reference only — **these indices are worthless on any other
build**. The extractor re-derives them from scratch every run.

| Role | Index |
|------|-------|
| u32 writer | `220` |
| u16 writer | `737` |
| u8 writer | `741` |
| u64 writer | `848` |
| fixslice bitslice | `645` |
| AES core | `237` |
| hook import | `a.Y (func 50)` |

## HSJ keys

`hsj.js` is the pure-JavaScript build of the same module, served instead of the
WASM one to clients that cannot run it. It uses the **same two keys**, and that
is measured rather than assumed — they are extracted from `hsj` and confirmed
through `hsj`'s own crypto, with no reference to the WASM side:

~~~
request_key  K  is accepted iff  GCM-open(K, hsj(1, pt))          == pt
response_key K  is accepted iff  hsj(0, iv || GCM(K,iv,pt) || tag) == pt
~~~

| Key | Value | Same as hsw |
|-----|-------|-------------|
| request\_key | `84dff2c4ab5b6c657a7ecfb6b0436243972729c346265f131b2a7d9822fd07c8` | yes |
| response\_key | `a4c213bc97c2fa8c59b7d96f453762115d8b45d680cad0408b12abe7ade9af9b` | yes |

So one keyring covers a build whichever bundle a client is served.

## History

Every build's confirmed keys are archived in [data/history/](data/history/).

Every entry has been confirmed against its own build's module. Earlier
unconfirmed values that were once published here have been replaced, not
amended — if an entry is present, its keys passed the checks above.

## License

MIT
