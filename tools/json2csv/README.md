# json2csv — JSON array of flat objects to CSV

**CLI:** `json2csv` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

Converts a JSON array of flat objects into CSV. The header is taken from the
first object's keys in source order (`json.Doc` preserves order); values in
later rows are matched by key, missing keys become empty cells, and extra
keys are ignored.

## CLI

```sh
echo '[{"name":"ada","id":7,"ok":true}]' | json2csv
# name,id,ok
# ada,7,true
json2csv data.json
```

Scalars render as text (strings unquoted, bools `true`/`false`, null empty,
numbers exactly as written in the source). Cells containing `,` `"` CR or LF
are quoted with doubled quotes (RFC 4180 style).

## Library

| Item | Meaning |
| --- | --- |
| `to_csv(json_text) -> Result<string, string>` | Full conversion; rejects non-array roots, non-object rows and nested containers |
| `csv_cell(cell) -> string` | Quote one cell when needed (`,` `"` CR LF trigger quoting, `"` doubles) |
| `scalar_cell(&Doc, node) -> Result<string, string>` | Render one scalar node as CSV text |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.json` | `Doc::parse_text`, `kind`/`count`/`key_at`/`val_at`/`field`, `as_str`/`as_bool`, `raw_span` |
| `std.bytes` | Cell quoting via a byte buffer |

No capabilities required — pure computation, reads stdin or a file via `sys`.

Run the tests: `dunder test`.
