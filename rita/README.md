# rita — ASCII charts

**CLI:** `rita` · **Library:** `import rita` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Rita* is Swedish for "draw". Terminal charts for numeric data — bars,
histograms, and sparklines rendered as plain text.

## Purpose

See the shape of numbers without leaving the terminal:

- Horizontal bars with value labels (`bar`)
- Histogram with automatic or fixed bucketing (`hist`)
- Sparklines for series data (`spark`)
- Pure formatting: no graphics, no filesystem — renders into a `string`

## CLI (v0.1.0)

Numbers are taken from the command line; each argument may mix separators
(spaces, tabs, commas, newlines):

```sh
rita bar 12 45 8 91
rita bar --width 6 1 2
rita hist --buckets 3 --width 10 1 2 2 3 3 3 9
rita spark 3 9 4 12 7 5 11 2
```

```text
$ rita bar 12 45 8 91
12.0 |█████
45.0 |████████████████████
 8.0 |████
91.0 |████████████████████████████████████████

$ rita spark 3 9 4 12 7 5 11 2
▂▆▂█▅▃▇▁
```

Histogram bucket edges are trimmed to at most two decimals (`1..3.67`, not
`3.6666666666666665`). Reading numbers from files/stdin and JSONL `--field`
input are planned for v0.2.

## Library

```dunder
import rita

fn main() -> i32 {
    parsed := rita.parse_numbers("3 9 4 12 7 5 11 2")
    match parsed {
        .Ok(values) => {
            opts := rita.Options { width: 40, buckets: 8, labels: false }
            print(rita.bar_chart(&values, &opts))
            print(rita.sparkline(&values))
        },
        .Err(e) => { print(e) },
    }
    return 0
}
```

| Item | Meaning |
| --- | --- |
| `Options` | Width, buckets, label mode |
| `parse_numbers(text) -> Result<[]f64, string>` | Args/CSV/whitespace → finite numbers |
| `min_value` / `max_value(&[]f64) -> f64` | Series bounds |
| `bar_chart(&[]f64, &Options) -> string` | Labeled horizontal bars |
| `histogram(&[]f64, &Options) -> string` | Bucketed counts with range labels |
| `sparkline(&[]f64) -> string` | Single-line block-character series |
| `format_edge(f64) -> string` | Two-decimal label trimming |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.parse` | `parse.value<f64>` |
| `std.bytes` | Buffer building with `append_utf8` / `decode` (block glyphs never split) |
| `f64` formatting | Shortest-round-trip `.str()`, `is_finite` checks |

No capabilities required. Computation only — a candidate for wasm32 builds.

## Dogfooding value

- f64 formatting, rounding edge cases and equal-value series (0/0 guards)
- Rune-level string building with multi-byte blocks
- Closes the suite loop once `salva` emits reports: `… | rita hist`

Run the tests: `dunder test` (12 suite tests + stdlib tests).
