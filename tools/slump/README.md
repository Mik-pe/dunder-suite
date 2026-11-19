# slump — random generator

**CLI:** `slump` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Slump* is Swedish for "random". Cryptographic random values as hex
strings, uniform integers in an inclusive range, or a pick among
arguments. `random_range` draws 7 bytes (56 bits, comfortably inside
i64) and scales them into the range — rejection-free and without the
modulo bias a naive `raw % span` would introduce.

## CLI

```sh
slump hex                  # 32 hex chars (16 bytes default)
slump hex --bytes 4        # -> db3cbbfc
slump int 1 100            # -> 42
slump pick apple banana cherry   # -> cherry
```

## Library

```dunder
import slump

fn main() -> i32 {
    match slump.random_range(1, 6) {
        .Ok(v) => { print(v.str()) },
        .Err(e) => { print(e) },
    }
    return 0
}
```

| Item | Meaning |
| --- | --- |
| `random_hex(n: i32) -> Result<string, string>` | n crypto-random bytes as hex (n 1..1024) via `crypto.random` + `bytes.hex_u8` |
| `random_range(min, max: i64) -> Result<i64, string>` | Uniform integer in `min..=max` (inclusive); rejects min > max |
| `random_index(len: i32) -> Result<i32, string>` | Uniform index 0..len-1 |

## Dunder capabilities used

`std.crypto` (`crypto.random`) and `std.bytes` (`hex_u8`) — no
capability line required for crypto itself.

Run the tests: `dunder test` (7 suite tests + stdlib tests).
