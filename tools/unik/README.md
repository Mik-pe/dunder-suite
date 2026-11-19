# unik — duplicate line collapser

**CLI:** `unik` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Unik* is Swedish for "unique". A uniq-style filter that collapses runs of
consecutive duplicate lines into one, optionally with a repeat count. Lines
must already be adjacent — sort first if you want global uniqueness.

## CLI

```sh
unik [-c] [FILE]
```

```text
$ printf 'a\na\nb\na\n' | unik -c -
      2 a
      1 b
      1 a
```

`-c` prefixes each output line with its count, right-aligned in 7 columns
(like uniq -c). `-` or a missing FILE reads standard input.
Exit: 0 ok, 1 read error, 2 usage.

## Library

| Item | Meaning |
| --- | --- |
| `Pair { text, count }` | One collapsed run |
| `collapse(&[]string) -> []Pair` | Merge consecutive equal lines |
| `render(&[]Pair, show_counts) -> string` | Joined output, with counts when asked |
| `to_lines(content) -> []string` | Split on `\n`, dropping one trailing empty part |

## Dunder capabilities used

None — `sys`, `std.text`.

Run the tests: `dunder test`.
