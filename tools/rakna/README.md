# rakna — integer arithmetic expression evaluator

**CLI:** `rakna` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Rakna* is Swedish for "count". Evaluates integer arithmetic expressions
with `+ - * / %`, proper operator precedence, parentheses and unary
minus, via a small recursive-descent parser that skips spaces between
tokens. Division by zero is an error (exit 1).

## CLI

Takes exactly one expression argument:

```sh
rakna "2 + 3 * (4 - 1)"      # -> 11
rakna "10 % 3 + 7 / 2"       # -> 4
rakna "-(2 + 3) * 4"         # -> -20
rakna "1 / 0"                # -> rakna: division by zero (exit 1)
```

## Library

```dunder
import rakna

fn main() -> i32 {
    match rakna.eval("2 + 3 * (4 - 1)") {
        .Ok(v) => { print(v.str()) },        // 11
        .Err(e) => { print(e) },
    }
    return 0
}
```

| Item | Meaning |
| --- | --- |
| `eval(expr: string) -> Result<i64, string>` | Evaluate `+ - * / %` with precedence, parens, unary minus; rejects garbage, unbalanced parens and division/modulo by zero |

## Dunder capabilities used

None — pure computation over strings and integers (`std` only implicitly).

Run the tests: `dunder test` (8 suite tests + stdlib tests).
