# prim — prime utilities

**CLI:** `prim` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

Prime questions for i64 integers using trial division: primality
checks, prime factorization with exponent notation, the Nth prime
(1-based) and listings of the first N primes.

## CLI

```sh
prim check 97        # -> prime
prim check 1         # -> prim: N must be >= 2 (exit 1)
prim factor 360      # -> 2^3 * 3^2 * 5
prim nth 10          # -> 29
prim list 5          # -> 2 3 5 7 11
```

`check`/`factor` reject N < 2 with a clear error; all subcommands cap
N at 10000 to keep trial-division runtimes sane.

## Library

```dunder
import prim

fn main() -> i32 {
    factors := prim.factorize(360)
    print(prim.format_factors(&factors))    // 2^3 * 3^2 * 5
    return 0
}
```

| Item | Meaning |
| --- | --- |
| `is_prime(n: i64) -> bool` | Trial division by 2 and odd i with i*i <= n |
| `factorize(n: i64) -> []i64` | Prime factors sorted ascending with repeats (empty for n < 2) |
| `format_factors(&[]i64) -> string` | `2^3 * 3 * 5` style rendering |
| `nth_prime(n: i32) -> i64` | Nth prime, 1-based (`nth_prime(1) == 2`); -1 for n < 1 |
| `first_primes(n: i32) -> []i64` | The first N primes in order |

## Dunder capabilities used

None — pure integer computation (`std.array` for joining lists).

Run the tests: `dunder test` (8 suite tests + stdlib tests).
