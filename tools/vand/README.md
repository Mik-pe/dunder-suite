# vand — reverser

**CLI:** `vand` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Vand* is Swedish for "turn". Reverses line order by default; with `-c` it
reverses the characters of each line instead, keeping the order. Character
reversal works on runes, not bytes, so multi-byte text survives intact.

## CLI

```sh
vand [-c] [FILE]
```

```text
$ printf 'first\nåäö line\n🌍x\n' | vand -c -
tsrif
enil öäå
x🌍
```

`-c` reverse runes per line; default reverses line order. `-` or a missing
FILE reads standard input. Exit: 0 ok, 1 read error, 2 usage.

## Library

| Item | Meaning |
| --- | --- |
| `reverse_lines(&[]string) -> []string` | Flip the order of the lines |
| `reverse_runes(s) -> string` | Reverse the runes of one string (`"åäö"` → `"öäå"`, `"🌍x"` → `"x🌍"`) |
| `to_lines(content) -> []string` | Split on `\n`, dropping one trailing empty part |

## Dunder capabilities used

None — `sys`, `std.text`.

Run the tests: `dunder test`.
