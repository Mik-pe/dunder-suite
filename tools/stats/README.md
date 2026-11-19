# stats — summary statistics

**CLI:** `stats` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

Summary statistics for a series of numbers: count, min, max, mean,
median (average of the middle two for even counts) and population
standard deviation. f64 values are rounded to two decimals for display
using integer math on v*100 (the `rita` `format_edge` pattern).

## CLI

Numbers come from arguments, or from stdin when no numbers are given;
separators may be spaces, tabs, newlines or commas:

```sh
stats 12 45 8 91
echo "12 45,8 91" | stats
```

```text
$ stats 12 45 8 91
count   4
min     8
max     91
mean    39
median  28.5
stdev   33.28
```

## Library

```dunder
import stats

fn main() -> i32 {
    match stats.parse_numbers("3 9 4 12") {
        .Ok(values) => {
            report := stats.summarize(&values)
            print(stats.render(&report))
        },
        .Err(e) => { print(e) },
    }
    return 0
}
```

| Item | Meaning |
| --- | --- |
| `parse_numbers(input: string) -> Result<[]f64, string>` | Whitespace/comma separated text to finite numbers |
| `summarize(&[]f64) -> Report` | Sorted-series statistics (uses `array.sort_by` with `math.total_cmp_f64`) |
| `render(&Report) -> string` | Aligned two-decimal report block |
| `format_2(f64) -> string` | Two-decimal display trimming |
| `Report` | `count, min, max, mean, median, stdev` |

## Dunder capabilities used

None for computation. Reading stdin uses `sys.read_text("/dev/stdin")`
(no capability line required for `sys` reads).

Run the tests: `dunder test` (8 suite tests + stdlib tests).
