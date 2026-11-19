# b64 — base64 encoder/decoder

**CLI:** `b64` · **Library:** `import b64` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

`b64` encodes and decodes standard base64 (RFC 4648 §4, alphabet
`A-Za-z0-9+/`, `=` padding). The codec is implemented by hand on byte arrays —
the Dunder 0.5.0 stdlib has no base64 module.

## CLI

```sh
echo -n hello | b64 encode          # stdin -> aGVsbG8=
b64 encode "Man"                    # data as argument -> TWFu
b64 encode photo.bin                # file (binary-safe) -> one base64 line
echo -n TWFu | b64 decode           # stdin -> Man
b64 decode TQ==                     # -> M
b64 decode TW!u                     # -> error, exit 1
```

After the subcommand, a single optional argument is accepted: `-` or nothing
reads stdin; an argument naming an existing file reads that file
(binary-safe); anything else is used as literal data. Decoding is strict:
invalid characters, wrong length, or misplaced `=` padding are rejected.
Decoded output must be valid UTF-8 to reach stdout (raw binary output is not
supported in v0.1).

## Library

| Item | Meaning |
| --- | --- |
| `encode(&[]u8) -> string` | Standard alphabet + `=` padding, 1/2/3-byte tails handled |
| `decode(string) -> Result<[]u8, string>` | Strict decode: length must be a multiple of 4, alphabet chars only, `=` only at the end of the last group |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.bytes`, `std.text` | byte-space work, trimming CLI input |
| `sys` | stdin/file reads (`read_u8`), metadata sniffing — no capability line required |

Run the tests: `dunder test`.
