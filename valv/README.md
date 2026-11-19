# valv — encrypted secrets vault

**CLI:** `valv` · **Library:** `import valv` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Valv* is Swedish for "vault". A local, passphrase-encrypted store for
credentials and tokens, built on `std.crypto`'s AES-256-GCM and Argon2id.

## Purpose

Keep machine secrets out of plaintext dotfiles:

- One vault file (default `./.valv`, `--file` to relocate)
- Argon2id verifies the passphrase; AES-256-GCM encrypts the payload
- Every save uses a fresh random nonce; tampering fails closed
- `gen` stores random hex tokens for deployments

## File format (v0.1.0)

```text
VALV1
$argon2id$v=19$m=19456,t=2,p=1$c2FsdA...    ← Argon2id PHC (passphrase verifier)
<hex nonce, 12 bytes>
<hex ciphertext of the JSON secrets>
<hex tag, 16 bytes>
```

Key derivation: the stored PHC string (which contains its own random salt)
is the input key material for HKDF-SHA256 → AES-256 key. Re-derivation is
deterministic because the PHC is stored; rotating the passphrase re-encrypts.

## CLI (v0.1.0)

Passphrases and secrets arrive on stdin so CI never needs a terminal:

```sh
printf 'pass\npass\n'   | valv init --file ~/.valv
printf 'pass\nsecret\n' | valv set api-key --file ~/.valv
printf 'pass\n'         | valv get api-key --file ~/.valv
printf 'pass\n'         | valv list --file ~/.valv
printf 'pass\n'         | valv rm api-key --file ~/.valv
printf 'pass\n'         | valv gen deploy-token --bytes 16 --file ~/.valv
printf 'old\nnew\nnew\n'| valv rotate --file ~/.valv
```

## Library

| Item | Meaning |
| --- | --- |
| `create(path, passphrase)` | New vault: Argon2id hash → HKDF key → sealed empty store |
| `open(path, passphrase)` | Verifies the passphrase, opens AES-GCM, decodes JSON entries |
| `store.get/set/remove/keys/len` | Owned, sorted access to secrets |
| `save(&store)` | Re-seals with a fresh nonce and rewrites the file |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.crypto` | `argon2id_hash`/`verify`, `hkdf_sha256`, `aes256_gcm_seal`/`open`, `random` |
| `std.json` / `std.map` | Payload encoding and entry storage |
| `std.bytes` | Hex encoding (`hex_u8`) and decoding (`crypto.decode_hex`) |

Requires `capability filesystem`. macOS needs Homebrew `openssl@3` + `argon2`;
Linux uses `libcrypto.so.3` + `libargon2.so.1` (as documented by `std.crypto`).

## Dogfooding value

- Correctness-critical FFI: nonces never reused, failures never leak plaintext
- The test suite proves wrong-passphrase and tampered-ciphertext rejection

Run the tests: `dunder test` (5 suite tests + stdlib tests).
