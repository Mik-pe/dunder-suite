# fyll — pad and truncate

**CLI:** `fyll` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Fyll* is Swedish for "fill". Pads each line to a target width — left-aligned
by default, right or center on request — or truncates lines that exceed it.
Widths are measured in runes, not bytes, so `åäö` counts as 3.

## CLI

```sh
fyll [-w N] [-m left|right|center] [-t] [FILE]
```

```text
$ printf 'ab\nåäö\nlong line here\n' | fyll -w 8 -m center -t -
   ab
  åäö
long lin
```

`-w N` width in runes (default 20); `-m` alignment (default left; center
splits the remainder, extra space on the right); `-t` truncate lines longer
than the width instead of leaving them. `-` or a missing FILE reads standard
input. Exit: 0 ok, 1 read error, 2 usage.

## Library

| Item | Meaning |
| --- | --- |
| `pad_line(s, width, mode, truncate) -> string` | Pad/truncate one line, rune-aware |
| `rune_len(s) -> i32` | Length in runes |
| `to_lines(content) -> []string` | Split on `\n`, dropping one trailing empty part |

## Dunder capabilities used

None — `sys`, `std.parse`, `std.text`.

Run the tests: `dunder test`.
