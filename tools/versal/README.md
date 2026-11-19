# versal — case conversion

**CLI:** `versal` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Versal* is Swedish for "capital letter". Converts ASCII letter case: upper
(default), lower, or title (first letter of each word capitalised, the rest
lowercased). Only ASCII `a-z`/`A-Z` are folded — non-ASCII letters such as
åäö pass through unchanged, byte for byte.

## CLI

```sh
versal [-m upper|lower|title] [FILE]
```

```text
$ printf 'hello åäö\nWORLD\n' | versal -m title -
Hello åäö
World
```

Words in title mode start after any non-alphanumeric ASCII character, so
`a-b c2d` becomes `A-B C2d`. `-` or a missing FILE reads standard input.
Exit: 0 ok, 1 read error, 2 usage.

## Library

| Item | Meaning |
| --- | --- |
| `upper_ascii(s) -> string` | Uppercase ASCII letters, pass everything else through |
| `lower_ascii(s) -> string` | Lowercase ASCII letters, pass everything else through |
| `title_ascii(s) -> string` | Capitalise the first ASCII letter of each word, lowercase the rest |
| `to_lines(content) -> []string` | Split on `\n`, dropping one trailing empty part |

## Dunder capabilities used

None — `sys`, `std.parse`, `std.text`, `std.bytes` (byte buffers for rebuilding strings).

Run the tests: `dunder test`.
