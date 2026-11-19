# radbryt — word wrap

**CLI:** `radbryt` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Radbryt* is Swedish for "line break". Wraps text to a maximum width (default
72 columns, measured in runes). Text is grouped into paragraphs separated by
blank lines; each paragraph is re-wrapped on spaces, and words longer than the
width are placed on their own line unbroken.

## CLI

```sh
radbryt [-w N] [FILE]
```

```text
$ printf 'The quick brown fox jumps over the lazy dog and keeps running.\n' | radbryt -w 30 -
The quick brown fox jumps over
the lazy dog and keeps running.
```

Paragraph breaks (blank lines) are preserved as a single empty line.
Consecutive non-blank lines are joined into one paragraph, fmt-style.
`-` or a missing FILE reads standard input. Exit: 0 ok, 1 read error, 2 usage.

## Library

| Item | Meaning |
| --- | --- |
| `wrap_text(text, width) -> string` | Wrap paragraphs to `width` runes |
| `to_lines(content) -> []string` | Split on `\n`, dropping one trailing empty part |

## Dunder capabilities used

None — `sys`, `std.parse`, `std.text`.

Run the tests: `dunder test`.
