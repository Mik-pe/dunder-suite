# packa — JSON minifier

**CLI:** `packa` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Packa* is Swedish for "pack". Parses JSON and re-emits it compact: no
whitespace between tokens, scalars kept exactly as written (numbers via
`raw_span`, strings re-quoted with `json.quote`).

## CLI

```sh
echo '{ "a" : 1, "b" : [ 1, 2 ] }' | packa
# {"a":1,"b":[1,2]}
packa one.json two.json     # one minified document per line
```

Invalid input reports `<path>: <parse error> at byte <offset>` on stderr and
the tool exits 1 after processing every file.

## Library

| Item | Meaning |
| --- | --- |
| `minify(json_text) -> Result<string, string>` | Parse + compact re-emit; numbers/bools/null via `raw_span`, strings via `json.quote` |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.json` | `Doc::parse_text`, `kind`/`count`/`key_at`/`val_at`/`at`, `as_str`, `raw_span`, `quote` |

No capabilities required — pure computation, reads stdin or files via `sys`.

Run the tests: `dunder test`.
