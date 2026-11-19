# snygga — JSON pretty-printer with options

**CLI:** `snygga` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Snygg* is Swedish for "sleek". Pretty-prints JSON with a configurable
indent (default 2 spaces). `--compact-flat` keeps arrays of scalars inline
(`[1, 2, 3]`) while objects — and arrays containing containers — stay
multi-line, like `jq`'s default style.

## CLI

```sh
echo '{"a":[1,2],"b":{"c":true}}' | snygga
# {
#   "a": [
#     1,
#     2
#   ],
#   "b": {
#     "c": true
#   }
# }
echo '{"a":[1,2],"b":{"c":true}}' | snygga --compact-flat
# {
#   "a": [1, 2],
#   "b": {
#     "c": true
#   }
# }
snygga -i 4 big.json
```

## Library

| Item | Meaning |
| --- | --- |
| `pretty(json_text, indent: i32, flat_arrays: bool) -> Result<string, string>` | Parse + pretty render; indent 0..16, `flat_arrays` inlines all-scalar arrays |

Strings print via `json.quote` of `as_str` (never `raw_span`, which is
unquoted for strings); numbers/bools/null keep their source text.

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.json` | `Doc::parse_text`, `kind`/`count`/`key_at`/`val_at`/`at`, `as_str`, `raw_span`, `quote` |
| `std.bytes` | Indented output buffer |

No capabilities required — pure computation, reads stdin or a file via `sys`.

Run the tests: `dunder test`.
