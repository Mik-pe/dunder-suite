# urlkod — percent-encoding filter

**CLI:** `urlkod` · **Library:** `import urlkod` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

`urlkod` percent-encodes and decodes text for URLs (RFC 3986). Encoding
operates on UTF-8 bytes: unreserved characters (`A-Za-z0-9-._~`) survive and
everything else becomes uppercase `%XX`. Decoding understands both hex cases
and form-style `+` as space, and rejects malformed escapes.

## CLI

```sh
echo -n "a b&c" | urlkod encode        # a%20b%26c
urlkod encode "mön"                    # m%C3%B6n
urlkod encode --full "a-b.c"           # a%2Db%2Ec   (only alphanumerics kept)
echo -n "a%2Bb%3Fc" | urlkod decode    # a+b?c
urlkod decode "a+b"                    # a b
urlkod decode "%ZZ"                    # error, exit 1
```

After the subcommand, one optional argument: `-` or nothing reads stdin, an
argument naming an existing file reads that file, anything else is literal
data.

## Library

| Item | Meaning |
| --- | --- |
| `encode(s: string, full: bool) -> string` | Keep unreserved (or with `full`, only alphanumerics); percent-encode the rest as uppercase `%XX` |
| `decode(s: string) -> Result<string, string>` | `%XX` (either hex case) and `+`→space; errors on lone `%`, bad hex, truncated escapes, or non-UTF-8 results |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.bytes`, `std.text` | byte-space work, CLI input trimming |
| `sys` | stdin/file reads, metadata sniffing — no capability line required |

Run the tests: `dunder test`.
