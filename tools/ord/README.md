# ord — line/word/byte counter

**CLI:** `ord` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Ord* is Swedish for "word". A wc-style counter: for each file it prints the
number of lines, words (runs of non-whitespace separated by ASCII whitespace)
and bytes, plus a total when given two or more files.

## CLI

```sh
ord FILE...        # one summary line per file, then "total" for 2+ files
ord -              # read standard input
```

```text
$ ord a.txt b.txt
      2       3      14 a.txt
      1       1       2 b.txt
      3       4      16 total
```

A line count is the number of `\n` bytes, plus one if the content is non-empty
and does not end with a newline. Unreadable files are reported on stderr and
set exit code 1; other files are still counted. Exit: 0 ok, 1 read error,
2 usage.

## Library

| Item | Meaning |
| --- | --- |
| `Counts { lines, words, bytes }` | One summary |
| `count_text(content) -> Counts` | Pure counting over a string |
| `render(&Counts, label) -> string` | The `f"{lines:7} {words:7} {bytes:7} label"` line |
| `add(&mut Counts, &Counts)` | Accumulate totals |

## Dunder capabilities used

None — `sys` only (file reads, args, stderr).

Run the tests: `dunder test`.
