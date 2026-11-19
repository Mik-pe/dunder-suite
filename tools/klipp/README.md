# klipp — field extractor

**CLI:** `klipp` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Klipp* is Swedish for "cut". A cut-style extractor that prints field N
(1-based) of every line, splitting on a separator (default: tab). Lines with
fewer than N fields print as empty lines, like cut.

## CLI

```sh
klipp [-d SEP] [-f N] [FILE]
```

```text
$ printf 'name\tage\tcity\nada\t42\n' | klipp -f 3
city

$ printf 'a,b,c\nx,y\n' | klipp -d , -f 2 -
b
y
```

`-d SEP` any string separator (default `\t`); `-f N` 1-based field number
(default 1). `-` or a missing FILE reads standard input. Exit: 0 ok, 1 read
error, 2 usage.

## Library

| Item | Meaning |
| --- | --- |
| `field(line, sep, n) -> Result<string, string>` | Field `n` of `line`, or an error when `n` is out of range / `sep` empty |
| `to_lines(content) -> []string` | Split on `\n`, dropping one trailing empty part |

## Dunder capabilities used

None — `sys`, `std.parse`, `std.text`.

Run the tests: `dunder test`.
