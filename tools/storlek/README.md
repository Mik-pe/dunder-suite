# storlek — recursive directory sizes

**CLI:** `storlek` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Storlek* is Swedish for "size" — it measures how many bytes live under each
top-level entry of one or more directories and ranks them largest-first.

## CLI

```sh
storlek                  # top 10 entries under .
storlek -s 3 src/ tests/ # top 3 across two roots
storlek file.txt         # a file argument measures just itself
```

```text
$ storlek -s 3 src/ tests/
        2368  src/main.ddr
        2078  tests/suite.ddr
        1178  src/storlek.ddr
        5624  TOTAL
```

Rows are `<bytes right-aligned 12>  <path>`; sizes count regular files
recursively (hidden entries included), ties break by path, and `TOTAL` sums
every measured entry, not just the ones shown.

## Library

| Item | Meaning |
| --- | --- |
| `SizeRow { path, bytes }` | One measured entry |
| `tree_size(path) -> i64` | Recursive byte total; unreadable paths contribute 0 |
| `format_row(bytes, path) -> string` | Right-aligned row rendering |
| `rank(&[]SizeRow, limit) -> []SizeRow` | Largest first, ties by path, truncated to `limit` |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.fs` | `list_dir` — `capability filesystem` |
| `sys` | `metadata` size/kind |
| `std.array` | `sort_by` with an `Ordering` comparator |

Run the tests: `dunder test`.
