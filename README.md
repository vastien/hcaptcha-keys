# hCaptcha HSW Key Tracker

> The two AES-256-GCM keys of each hCaptcha `hsw.js` build, extracted from the
> build's own WASM module and confirmed against it before being published.

![Last checked](https://img.shields.io/badge/last_checked-2026-10-22_13%3A58%3A26_UTC-blue)
![Last updated](https://img.shields.io/badge/last_updated-2026-10-22_13%3A58%3A26_UTC-green)
![Build](https://img.shields.io/badge/build-e649efc4744281a6-lightgrey)
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
| **Build** | `e649efc4744281a6301a95d6d9ff8f7da1e0d2188c4a666eb09929ccc7d7277d` |
| **Bundle** | `https://newassets.hcaptcha.com/c/e649efc4744281a6301a95d6d9ff8f7da1e0d2188c4a666eb09929ccc7d7277d/hsw.js` |
| **Bundle SRI** | `sha256-uCuKwDizFLJLxLj1sTupebD98gL0hEC+ZZHsRdoINAU=` |
| **Runtime WASM sha256** | `1e4b64e0a0ad7ceee5758e40cdc6c334bd2cf0c6eb23dfc9d623e52f9dcbe6a9` |
| **Last checked** | 2026-10-22 13:58:26 UTC |
| **Last updated** | 2026-10-22 13:58:26 UTC |

## HSW keys

Cipher: **AES-256-GCM**. Wire format both ways:
`iv(12) || ciphertext(N) || tag(16)`.

| Key | Value | What it is for |
|-----|-------|----------------|
| request\_key | `7155f6ecfabc22c5a57d24657de65f397f71b89b25a968e18e28b2360dd944fa` | Encrypts the outbound request body — the key `hsw(1, plaintext)` uses |
| response\_key | `e93abd674eea48703329af233ad4c9d08d9c9a2e11c1547747fe39963317ced9` | Decrypts the `/getcaptcha` response — the key `hsw(0, blob)` uses |

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
`fingerprint_blob_key`. Both have been removed: they were wrong. The n-token is
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
| u32 writer | `734` |
| u16 writer | `691` |
| u8 writer | `1041` |
| u64 writer | `524` |
| fixslice bitslice | `234` |
| AES core | `1081` |
| hook import | `a.r (func 17)` |

## Running the extractor yourself

The extractor lives in [vastien/hcaptcha-keys-tracker](https://github.com/vastien/hcaptcha-keys-tracker).
It needs Node 18+ and [wabt](https://github.com/WebAssembly/wabt) (`wasm2wat`, `wat2wasm`).

```bash
node src/index.js                                   # the build hCaptcha is serving now
node src/index.js --version <64-hex> --history-only  # backfill an older build
node src/index.js --dry-run                          # extract and print, write nothing
node test/regression.js                              # check it against known builds
```

A run takes about five seconds. The full method, and the traps that each cost
hours, are written up in
[METHOD.md](https://github.com/vastien/hcaptcha-keys-tracker/blob/main/METHOD.md).

### How the keys are found, in short

1. Fetch `hsw.js`. The base64 WASM blob inside it is a ~60 KB stub — the real
   ~980 KB module is assembled at runtime, so the bundle is loaded in plain Node
   over a small mocked browser surface and `WebAssembly.instantiate` is hooked to
   dump the module. That also leaves `hsw(0,..)` and `hsw(1,..)` callable, which
   is where the oracle comes from.
2. Disassemble, and find this build's functions **structurally** — never by
   index, since no index survives a rotation. Memory accessors are the functions
   containing both `i32.const 320; i32.div_u` and `i32.const 320; i32.rem_u`,
   classified by signature; the fixslice functions are the ones containing the
   bit-transposition constants `1431655765`, `858993459`, `252645135`,
   `-252645136`.
3. Patch the u32 writer to report every de-obfuscated `(addr, value)` on entry,
   and tag the `bitslice` function's entry so the write log can be segmented per
   invocation. Reporting goes through an existing `(i32,i32,i32)->()` import,
   tagged with a magic argument — adding an import would renumber every defined
   function and invalidate every `call N`.
4. Each `bitslice` invocation's first 8 words are one bitsliced 16-byte block.
   `inv_bitslice` on them returns two identical halves, and two successive such
   values are `key[0:16]` then `key[16:32]`.
5. Confirm against the module, and publish only what passes.

The key is never 32 contiguous bytes anywhere, and it never reaches linear
memory either — the scratch region is reused before the call returns, so
scanning the final memory image finds nothing. That is why interception, rather
than searching, is the only thing that works here.

## History

Every build's confirmed keys are archived in [data/history/](data/history/).

Entries from before this method worked carry a `retracted` block instead of
keys. Their values were never confirmed against the module and are wrong; they
are kept, out of the way, as a record of what was once published.

## License

MIT
