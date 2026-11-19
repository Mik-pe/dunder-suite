# omvandla — unit converter

**CLI:** `omvandla` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Omvandla* is Swedish for "convert". Converts a value between units of
the same category: temperature (c, f, k — affine, with offsets),
distance (km, mi, m, ft), weight (kg, lb, g) and data (b, kb, mb, gb —
decimal, 1000-based). Units are case-insensitive; mixing categories is
an error (`cannot convert kg to km`).

## CLI

```sh
omvandla 0 c f        # -> 32
omvandla 100 c f      # -> 212
omvandla 1 mi km      # -> 1.61
omvandla 1 kg km      # -> omvandla: cannot convert kg to km (exit 1)
```

## Library

```dunder
import omvandla

fn main() -> i32 {
    match omvandla.convert(1.0, "mi", "km") {
        .Ok(v) => { print(omvandla.format_2(v)) },    // 1.61
        .Err(e) => { print(e) },
    }
    return 0
}
```

| Item | Meaning |
| --- | --- |
| `convert(value: f64, from_unit: string, to_unit: string) -> Result<f64, string>` | Temperatures convert through Celsius; others through linear ratios (meters, grams, bytes) |
| `near(a, b: f64) -> bool` | Tolerance helper: both `a-b` and `b-a` < 0.001 |
| `format_2(f64) -> string` | Two-decimal display trimming |

## Dunder capabilities used

None — pure f64 computation (`std.parse` for argument parsing).

Run the tests: `dunder test` (10 suite tests + stdlib tests).
