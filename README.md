# hCaptcha HSW Key Tracker

> The two AES-256-GCM keys of each hCaptcha `hsw.js` build, extracted from the
> build's own WASM module and confirmed against it before being published — plus
> the AES-128-CBC key that build seals its `fingerprint_blob` with.

![Last checked](https://img.shields.io/badge/last_checked-2026-11-02_21%3A54%3A12_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-11-02_21%3A54%3A12_UTC-green)
![Build](https://img.shields.io/badge/build-080430368d24043a-lightgrey)
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
| **Build** | `080430368d24043aa49d4d84dd3dd99171556d0f7bb0afe45d8a8e50cf2a3ef7` |
| **Bundle** | `https://newassets.hcaptcha.com/c/080430368d24043aa49d4d84dd3dd99171556d0f7bb0afe45d8a8e50cf2a3ef7/hsw.js` |
| **Bundle SRI** | `sha256-217QPnFR+f9V53UoZG6AInlTyAdGuV13iRFk0vpwOd0=` |
| **Runtime WASM sha256** | `d48cb57c7d0a60f062038daf30b01467df51853a3a3a080bea3b6e5705f0cd75` |
| **Last checked** | 2026-11-02 21:54:12 UTC |
| **Last updated** | 2026-11-02 21:54:12 UTC |

## HSW keys

Cipher: **AES-256-GCM**. Wire format both ways:
`iv(12) || ciphertext(N) || tag(16)`.

| Key | Value | What it is for |
|-----|-------|----------------|
| request\_key | `78a66547dc6cdf89e065e7cd9f50cbd3a6c7f0d418e6f579b61c21e407d0a968` | Encrypts the outbound request body — the key `hsw(1, plaintext)` uses |
| response\_key | `59ff64bef406605a819b4fe9054077350285ca7b434687a846a72e7da5e91b07` | Decrypts the `/getcaptcha` response — the key `hsw(0, blob)` uses |

They are **two different keys**, and neither one works in place of the other.

### How each one is confirmed

Nothing is published unless the build's own module agrees with it:

~~~
request_key  K  is accepted iff  GCM-open(K, hsw(1, pt))          == pt
response_key K  is accepted iff  hsw(0, iv || GCM(K,iv,pt) || tag) == pt
~~~

A candidate that fails is discarded, and a build where neither key can be
confirmed fails the run and leaves the previous data untouched.

## The n-token has no key, but its `fingerprint_blob` does

The n-token itself is **not** encrypted under a recoverable key — it is a
deterministic pipeline (a 31-bit repack, five modular passes and a word
rotation, XORed against a fixed keystream with no key, nonce or per-call
entropy). The `n_key` earlier versions of this repository published was wrong
and stays removed.

One field *inside* the decoded payload is keyed, though: `fingerprint_blob`,
which the bundle seals separately before the payload is encoded. That key is
real, per build, and recoverable — see below.

## Blob keys (`fingerprint_blob`)

Cipher: **AES-128-CBC** with PKCS#7 padding, sealed in the bundle's plain
**JavaScript**, not in the WASM module. Wire format:

~~~
<base64 iv> "." <base64 ciphertext>      # IV is 16 bytes from crypto.getRandomValues
~~~

Plaintext is JSON: `[[event_id, value], ...]`, the behavioural event list.

| Build | Key |
|-------|-----|
| `425954adc4d740af` | `d9d0575a50d6aaad73b2c8b1e4714475` |
| `e649efc4744281a6` | `72a3ed76fa5cf8a95c030a623250fbc3` |

Full hashes and per-build detail are in [data/history/](data/history/), under a
`blob` key alongside `hsw` and `hsj`. A build whose blob key is known but whose
hsw/hsj keys were never captured gets a history entry with only the `blob`
section.

These keys are per build like every other key here, so this table needs a new
row every rotation — several a day. A blob key is not re-derived on a schedule
the way the hsw and hsj keys are: a row appears once that build's key has been
recovered and confirmed, and is carried forward from the archive.

### How each one is confirmed

`K` is accepted iff it opens a `fingerprint_blob` from a **separate** run of
the same build to a well-formed plaintext — valid PKCS#7 padding and a JSON
event list. A candidate that fails is discarded and nothing is published for
that build.

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

```javascript
// decrypt a fingerprint_blob: "<base64 iv>.<base64 ct>", AES-128-CBC
const blobKey = Buffer.from('d9d0575a50d6aaad73b2c8b1e4714475', 'hex');  // 16 bytes, build 425954ad
const [ivB64, ctB64] = fingerprintBlob.split('.');
const d2 = crypto.createDecipheriv('aes-128-cbc', blobKey, Buffer.from(ivB64, 'base64'));
const events = JSON.parse(
  Buffer.concat([d2.update(Buffer.from(ctB64, 'base64')), d2.final()]).toString()
);  // [[event_id, value], ...]
```

## This build's function mapping

Recorded for reference only — **these indices are worthless on any other
build**. The extractor re-derives them from scratch every run.

| Role | Index |
|------|-------|
| u32 writer | `777` |
| u16 writer | `1029` |
| u8 writer | `578` |
| u64 writer | `645` |
| fixslice bitslice | `529` |
| AES core | `838` |
| hook import | `a.Na (func 93)` |

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
| request\_key | `78a66547dc6cdf89e065e7cd9f50cbd3a6c7f0d418e6f579b61c21e407d0a968` | yes |
| response\_key | `59ff64bef406605a819b4fe9054077350285ca7b434687a846a72e7da5e91b07` | yes |

So one keyring covers a build whichever bundle a client is served.

## History

Every build's confirmed keys are archived in [data/history/](data/history/).

Every entry has been confirmed against its own build: the hsw/hsj keys against
its WASM module, a `blob` key against the bundle's own JavaScript. Earlier
unconfirmed values that were once published here have been replaced, not
amended — if an entry is present, its keys passed the checks above.

## License

MIT
