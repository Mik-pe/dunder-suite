# sortera — line sorter

**CLI:** `sortera` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Sortera* is Swedish for "sort". Sorts lines lexically or numerically, with
optional reverse and dedupe. In numeric mode lines that fail `f64` parsing
are collected separately and appended last in their original order.

## CLI

```sh
printf '9\n10\n2\n' | sortera -n
# 2
# 9
# 10
sortera -r names.txt
printf 'b\n9\nx\n10\n' | sortera -n      # unparseable lines go last:
# 9
# 10
# b
# x
```

Flags combine freely; `-r` reverses after sorting (numeric tail stays last),
`-u` dedupes adjacent equal lines after sorting.

## Library

| Item | Meaning |
| --- | --- |
| `sort_lines(&[]string, numeric, reverse, unique) -> []string` | Partition (numeric only) → `array.sort`/`sort_by` → dedupe → reverse → append tail |

Sorting uses a named comparator: `compare_numeric` compares via
`parse.value<f64>` (falling back to text compare) with
`math.total_cmp_f64`; plain mode uses UTF-8 byte order.

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.array` / `std.math` / `std.parse` | `sort`, `sort_by`, `reverse`, `total_cmp_f64`, `value<f64>` |

No capabilities required — pure computation, reads stdin or a file via `sys`.

Run the tests: `dunder test`.
