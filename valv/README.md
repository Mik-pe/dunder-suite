# valv — encrypted secrets vault

**CLI:** `valv` · **Library:** `import valv` · **Status:** planned · **Target:** Dunder 0.5.0

*Valv* is Swedish for "vault". A local, passphrase-encrypted store for
credentials and tokens, built on `std.crypto`'s AES-256-GCM and Argon2id.

## Purpose

Keep machine secrets out of plaintext dotfiles:

- One encrypted vault file (default `~/.valv/vault.valv`)
- Passphrase → Argon2id key derivation; every save uses a fresh random nonce
- Authenticated encryption: tampering fails closed, no plaintext on bad tags
- Add-on generation mode for tokens (`valv gen`) using `crypto.random`

## CLI

```sh
valv init                                # create vault, set passphrase
valv set api-key                         # prompted for the secret value
valv set api-key < token.txt             # or read the secret from stdin
valv get api-key                         # prints value (for scripts/CI)
valv list                                # keys + updated timestamps, no values
valv rm api-key                          # remove a key
valv gen deploy-token --length 32        # random hex token, stored and printed
valv rotate                              # new passphrase, re-encrypts in place
```

Passphrase comes from `VALV_PASSPHRASE` when set, otherwise a stdin prompt —
so CI jobs never need interactive input.

## Library

```dunder
import valv

fn main() -> i32 {
    opened := valv.Store::open("~/.valv/vault.valv")
    if !opened.ok { print(opened.error); return 1 }
    mut store := opened.store
    match store.get("api-key") {
        .Ok(secret) => { _ := sys.write_stderr(f"use {secret}") },
        .Err => { print("api-key not found") },
    }
    return 0
}
```

Planned API:

| Item | Meaning |
| --- | --- |
| `Store::open(path)` | OpenResult; verifies passphrase before decrypting the index |
| `store.get(key)` | `Result<string, valv.Error>`; constant-time comparisons where lengths allow |
| `store.set(move key, move value)` / `store.remove(key)` | Mutate + re-encrypt on `save()` |
| `store.save()` | Fresh `crypto.random` nonce per write, atomic replace of the file |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.crypto` | `argon2id_hash`/`verify` for the passphrase, `aes256_gcm_seal`/`open`, `random` for nonces and `gen`, `equal` for comparisons |
| `std.json` | Encrypted payload is a JSON object of key → secret, so padding stays opaque but the format is inspectable when decrypted |
| `sys` | `arg`, env lookup, `write_text`/`rename` for atomic saves |

Requires `capability filesystem`. macOS needs Homebrew `openssl@3` + `argon2`;
Linux uses `libcrypto.so.3` + `libargon2.so.1` (as documented by `std.crypto`).

## Dogfooding value

- Correctness-critical FFI usage: nonces never reused, errors never leak plaintext
- Byte-buffer handling (`[]u8` seal/open results) and secret lifecycle (zero reliance on string sharing)
- A real consumer of Argon2id + AES-GCM beyond compiler test fixtures
