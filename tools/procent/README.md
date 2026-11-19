# procent — percentage math

**CLI:** `procent` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

Percentage arithmetic: X% of Y, A as a percentage of B, and percent
change from A to B — all with two-decimal output. Division by zero
(and change from 0) are errors, not panics.

## CLI

```sh
procent av 25 200            # 25% of 200      -> 50
procent andel 3 4            # 3 is what % of 4 -> 75
procent forandring 100 150   # change 100 -> 150 -> 50
procent andel 1 0            # -> procent: cannot divide by zero (exit 1)
```

## Library

```dunder
import procent

fn main() -> i32 {
    match procent.change(100.0, 150.0) {
        .Ok(v) => { print(procent.format_2(v)) },    // 50
        .Err(e) => { print(e) },
    }
    return 0
}
```

| Item | Meaning |
| --- | --- |
| `percent_of(x: f64, y: f64) -> f64` | x% of y |
| `share(a, b: f64) -> Result<f64, string>` | a as a percentage of b; error on b = 0 |
| `change(a, b: f64) -> Result<f64, string>` | percent change from a to b; error on a = 0 |
| `format_2(f64) -> string` | Two-decimal display trimming |

## Dunder capabilities used

None — pure f64 computation (`std.parse` for argument parsing).

Run the tests: `dunder test` (6 suite tests + stdlib tests).
