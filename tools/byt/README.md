# byt — batch renamer

**CLI:** `byt` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Byt* is Swedish for "change/exchange" — it renames files whose *filename*
contains a piece of text, replacing its first occurrence.

## CLI

```sh
byt --dry -orig -edit photo-orig.jpg   # preview only
byt -v1 -final draft-v1.txt notes.txt  # rename for real
```

```text
$ byt --dry -orig -edit photo-orig.jpg
photo-orig.jpg -> photo-edit.jpg
```

Only the filename is inspected (directories in the path are untouched), the
first occurrence of OLD is replaced, and paths with no match are skipped
silently. Only words starting with `--` are flags, so OLD/NEW may begin with
`-` (as above). Failed renames print to stderr and set exit 1.

## Library

| Item | Meaning |
| --- | --- |
| `replace_in_filename(path, old, new) -> Result<string, string>` | New full path with the first `old` in the filename replaced by `new`; `.Ok("")` when the filename has no match; `.Err` when `old` is empty |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `sys` | `metadata`, `rename` — no capability needed |
| `std.text` | `find` + string slicing |

Run the tests: `dunder test`.
