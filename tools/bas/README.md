# bas — integer base converter

**CLI:** `bas` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

Converts integers between bases 2, 8, 10 and 16. Digits `0-9` and `a-f`
(case-insensitive on input); output is lowercase, and negative values
keep a leading `-`. Implemented with repeated multiplication/division on
the magnitude — no bit operations.

## CLI

```sh
bas --from 16 --to 10 ff     # -> 255
bas --to 2 255               # -> 11111111
bas --from 2 1010            # -> 10 (default --to 10)
bas --to 16 -255             # -> -ff
```

Defaults: `--from 10`, `--to 10`.

## Library

```dunder
import bas

fn main() -> i32 {
    match bas.parse_in_base("ff", 16) {
        .Ok(v) => { print(bas.format_base(v, 2)) },   // 11111111
        .Err(e) => { print(e) },
    }
    return 0
}
```

| Item | Meaning |
| --- | --- |
| `valid_base(i64) -> bool` | True for 2, 8, 10, 16 |
| `parse_in_base(s: string, base: i64) -> Result<i64, string>` | Digits (case-insensitive, optional `-`) to value; rejects invalid and out-of-range digits |
| `format_base(value: i64, base: i64) -> string` | Value to digits; negative values keep a leading `-` |

## Dunder capabilities used

None — pure computation (`std.text` for trimming).

Run the tests: `dunder test` (7 suite tests + stdlib tests).
