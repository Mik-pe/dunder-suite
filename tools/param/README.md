# param — URL query string ↔ JSON

**CLI:** `param` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

Converts between URL query strings and JSON objects. `parse` decodes `%XX`
escapes and `+` as space, turning repeated keys into a JSON array;
`build` reads a JSON object and percent-encodes everything outside the
unreserved set (`A-Z a-z 0-9 - . _ ~`), repeating the key for array values.

## CLI

```sh
param parse "a=1&b=x%20y"
# {
#   "a": "1",
#   "b": "x y"
# }
echo '{"a":"1","b":"x y","k":["1","2"]}' | param build
# a=1&b=x%20y&k=1&k=2
param build obj.json
```

## Library

| Item | Meaning |
| --- | --- |
| `parse_query(query) -> Result<string, string>` | Query string → pretty JSON object text (source key order, repeated keys → arrays) |
| `build_query(json_text) -> Result<string, string>` | JSON object → query string; string or string-array values only |
| `decode_percent(&string) -> Result<string, string>` | `%XX` + `+` decoding with byte-precise error positions |
| `encode_percent(&string) -> string` | `%XX` encoding (uppercase hex, unreserved set kept) |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.json` | `Doc::parse_text`, `key_at`/`val_at`/`at`/`kind`/`as_str`, `quote` |
| `std.text` / `std.bytes` | `&`-splitting, `=` location, byte buffers |

No capabilities required — pure computation, reads stdin or a file via `sys`.

Run the tests: `dunder test`.
