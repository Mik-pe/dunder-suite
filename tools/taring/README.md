# taring — dice roller

**CLI:** `taring` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Tärning* is Swedish for "die". Rolls dice expressions like `2d6+3`,
`d20` or `3d8-2` using the crypto random generator — 7 random bytes
(56 bits) scaled into `[1, sides]`, so no modulo bias.

## CLI

```sh
taring 2d6+3    # -> 9  (2 dice, 6 sides, +3)
taring d20      # -> 17
taring 3d8-2    # -> 13
```

Prints the roll total. Validation: sides 2..100000, count 1..1000,
modifier within ±1000000.

## Library

```dunder
import taring

fn main() -> i32 {
    match taring.parse_dice("2d6+3") {
        .Ok(dice) => {
            match taring.roll_detailed(&dice) {
                .Ok(r) => { print(r.rolls.len.str() + " dice, total " + r.total.str()) },
                .Err(e) => { print(e) },
            }
        },
        .Err(e) => { print(e) },
    }
    return 0
}
```

| Item | Meaning |
| --- | --- |
| `Dice` | `count, sides, modifier` |
| `RollResult` | `rolls: []i32, total: i32` |
| `parse_dice(expr: string) -> Result<Dice, string>` | Parse `NdM[+K\|-K]` (also bare `dM`, uppercase `D`) with validation |
| `total(rolls: &[]i32, modifier: i32) -> i32` | Pure sum of fixed rolls plus modifier |
| `roll_detailed(&Dice) -> Result<RollResult, string>` | Individual faces and the total |
| `roll(&Dice) -> Result<i64, string>` | Just the total |

## Dunder capabilities used

`std.crypto` (`crypto.random`) — no capability line required for crypto
itself.

Run the tests: `dunder test` (6 suite tests + stdlib tests).
