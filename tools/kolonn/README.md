# kolonn — aligned table from TSV

**CLI:** `kolonn` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Kolonn* is Swedish for "column". Reads tab-separated rows and renders them
as an aligned text table: 2-space gutters, every column as wide as its
longest cell, nothing truncated. Ragged rows are padded with empty cells; a
trailing empty line (the final newline) is ignored.

## CLI

```sh
printf 'name\tid\nada\t7\nlovelace\t42\n' | kolonn
# name     id
# ada      7
# lovelace  42
kolonn data.tsv
```

## Library

| Item | Meaning |
| --- | --- |
| `split_tsv(text) -> [][]string` | Split on lines (last empty line dropped) then tabs (`text.split` preserves empty fields, which is correct here) |
| `align(rows: &[][]string) -> string` | Width = max cell byte length per column, 2-space gutter, right-pad with spaces |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.text` / `std.bytes` | Line/tab splitting, padded output buffer |

No capabilities required — pure computation, reads stdin or a file via `sys`.

Run the tests: `dunder test`.
