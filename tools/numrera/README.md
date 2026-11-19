# numrera — line numbering

**CLI:** `numrera` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Numrera* is Swedish for "number". Prefixes each line with a right-aligned
line number, two spaces, then the original line — like `nl -ba` or `cat -n`.

## CLI

```sh
numrera [-s N] [-w W] [FILE]
```

```text
$ printf 'first\nsecond\nthird\n' | numrera -s 5 -w 3 -
  5  first
  6  second
  7  third
```

`-s N` first line number (default 1); `-w W` number field width in digits,
right-aligned (default 6). `-` or a missing FILE reads standard input.
Exit: 0 ok, 1 read error, 2 usage.

## Library

| Item | Meaning |
| --- | --- |
| `number_lines(&[]string, start, width) -> string` | Render numbered output for the lines |
| `pad_number(n, width) -> string` | Right-align one number with spaces |
| `to_lines(content) -> []string` | Split on `\n`, dropping one trailing empty part |

## Dunder capabilities used

None — `sys`, `std.parse`, `std.text`.

Run the tests: `dunder test`.
