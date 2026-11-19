# toml2json — TOML to pretty JSON

**CLI:** `toml2json` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

Converts TOML into pretty-printed JSON with 2-space indentation. Uses
`std.toml` for parsing and re-renders through `json.Doc` so key order is
preserved and strings are re-quoted with `json.quote` (never `raw_span`,
which returns unquoted text for string nodes).

## CLI

```sh
printf 'name = "dunder"\n[deps]\nhttp = "pkg"\n' | toml2json
# {
#   "name": "dunder",
#   "deps": {
#     "http": "pkg"
#   }
# }
toml2json ../dunder.toml
```

## Library

| Item | Meaning |
| --- | --- |
| `convert(text) -> Result<string, string>` | TOML text → pretty JSON; carries `toml.parse` errors through |
| `render_pretty(&Doc, node) -> string` | Two-space pretty renderer for any parsed JSON node (strings via `json.quote`) |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.toml` | `parse` → JSON text |
| `std.json` | `Doc::parse_text`, `kind`/`count`/`key_at`/`val_at`/`at`, `as_str`, `raw_span`, `quote` |
| `std.bytes` | Output buffer helpers |

No capabilities required — pure computation, reads stdin or a file via `sys`.

Run the tests: `dunder test`.
