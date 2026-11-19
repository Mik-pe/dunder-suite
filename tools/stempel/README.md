# stempel — HMAC-SHA256 signer

**CLI:** `stempel` · **Library:** `import stempel` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Stempel* is Swedish for "stamp". `stempel` stamps files with an HMAC-SHA256
signature under a key and verifies them in constant time. Output matches
`openssl dgst -sha256 -hmac` exactly.

## CLI

```sh
stempel KEY FILE                        # print 64 hex chars
stempel --verify KEY FILE HEX           # print OK/FAIL, exit 0/1
```

```text
$ printf 'payload\n' > msg.txt
$ stempel hemlig msg.txt
f316142aaa0bd2d7c2560901e6ccde19a6350cdc6a18d3689918e0f5d2e82105
$ stempel --verify hemlig msg.txt f316142a...e82105
stempel: OK
```

Verification decodes both HMACs to bytes and compares with `crypto.equal`
(constant-time). A non-hex expected value is an error (exit 1), not a
mismatch.

## Library

| Item | Meaning |
| --- | --- |
| `hmac_hex(key: string, path: string) -> Result<string, string>` | HMAC-SHA256(key-utf8-bytes, file-bytes) as lowercase hex |
| `verify_hex(key: string, path: string, expected: string) -> Result<bool, string>` | Constant-time comparison against a hex expectation; `Err` for bad hex or unreadable files |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.crypto` | `hmac_sha256`, `decode_hex`, `equal` |
| `sys` | raw file reads — no capability line required |
| `std.fs` (tests only) | `capability filesystem` for temp-dir fixtures |

Run the tests: `dunder test`.
