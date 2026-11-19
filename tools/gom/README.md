# gom — single-file AES-256-GCM encryptor

**CLI:** `gom` · **Library:** `import gom` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Gom* is Swedish for "gum/hide". `gom` hides one file in one encrypted file
using AES-256-GCM under a passphrase — a self-contained, simpler sibling of
the `valv` secret vault, reusing exactly its key-derivation scheme: the
argon2id PHC string feeds HKDF-SHA256 (salt `gom-v1`, info `gom-key`), and
the derived 32-byte key seals the payload with the ASCII string `GOM1` as
GCM additional-authenticated data.

## File format

```text
GOM1\n<argon2id PHC>\n<hex nonce 12>\n<hex ciphertext>\n<hex tag 16>\n
```

## CLI

```sh
gom encrypt IN OUT    # passphrase on stdin line 1, confirmation on line 2
gom decrypt IN OUT    # passphrase on stdin line 1
```

```text
$ printf 'hello\n' > note.txt
$ printf 'hemlighet\nhemlighet\n' | gom encrypt note.txt note.gom
$ printf 'hemlighet\n' | gom decrypt note.gom note.out
$ cat note.out
hello
$ printf 'fel\n' | gom decrypt note.gom note.out
gom: wrong passphrase    (exit 1)
```

The passphrase is verified with argon2id **and** the GCM tag: a wrong
passphrase or a single flipped byte in the file fails decryption. Inputs are
capped at 64 MiB.

## Library

| Item | Meaning |
| --- | --- |
| `encrypt_to(path_in, path_out, passphrase) -> Result<(), string>` | Seal the input file into a `GOM1` text file |
| `decrypt_to(path_in, path_out, passphrase) -> Result<(), string>` | Verify + open; writes the exact original bytes back out |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.crypto` | `argon2id_hash`/`verify`, `hkdf_sha256`, `random`, `aes256_gcm_seal`/`open` |
| `sys` | file I/O, stdin passphrase — no capability line required |
| `std.fs` (tests only) | `capability filesystem` for temp-dir fixtures |

Run the tests: `dunder test`.
