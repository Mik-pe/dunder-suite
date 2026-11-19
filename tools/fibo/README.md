# fibo — Fibonacci numbers

**CLI:** `fibo` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

Fibonacci numbers on i64 with F(1) = 1, F(2) = 1. Lists the first N
numbers or prints a single F(N); F(92) = 7540113804746346429 is the
largest Fibonacci number that fits in i64, so `nth` rejects N > 92
with a clear overflow error.

## CLI

```sh
fibo 10        # -> 1 1 2 3 5 8 13 21 34 55
fibo nth 10    # -> 55
fibo nth 92    # -> 7540113804746346429
fibo nth 93    # -> fibo: N must be <= 92 (F(93) overflows i64) (exit 1)
```

## Library

```dunder
import fibo

fn main() -> i32 {
    values := fibo.fibonacci(10)
    for v in values { print(v.str()) }
    return 0
}
```

| Item | Meaning |
| --- | --- |
| `fibonacci(count: i32) -> []i64` | First `count` Fibonacci numbers (capped at 92 entries) |
| `nth(n: i32) -> Result<i64, string>` | F(n), 1-based; error for n < 1 or n > 92 |

## Dunder capabilities used

None — pure i64 computation.

Run the tests: `dunder test` (6 suite tests + stdlib tests).
