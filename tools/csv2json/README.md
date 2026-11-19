# csv2json — CSV to JSON array of objects

**CLI:** `csv2json` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

Parses CSV (RFC 4180 style quoting) into a JSON array of objects. Without
`--header` the first data row names the columns; with `--header` every row is
a data row. Quoted cells may contain commas, doubled quotes and embedded
newlines.

## CLI

```sh
printf 'a,b\n1,"x,y"\n' | csv2json
# [{"a":"1","b":"x,y"}]
csv2json --header name,id people.csv
```

All values are emitted as JSON strings. Short rows emit fewer keys; cells
beyond the header are ignored.

## Library

| Item | Meaning |
| --- | --- |
| `parse_csv(text) -> Result<[][]string, string>` | Byte-level state machine: quoting, `""` escapes, CRLF/LF rows, ragged rows; precise errors |
| `to_json(rows: &[][]string, header: &[]string) -> string` | `json.Writer` rendering of data rows against the header |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.json` | `Writer` (`array`/`object`/`key`/`string`/`end`/`finish`) |
| `std.bytes` | Cell accumulation and UTF-8 decode |

No capabilities required — pure computation, reads stdin or a file via `sys`.

Run the tests: `dunder test`.
