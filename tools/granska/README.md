# granska — TODO/FIXME scanner

**CLI:** `granska` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Granska* is Swedish for "examine". Recursively scans `.ddr`, `.toml`, `.md`
and `.txt` files (hidden directories are skipped) for the markers TODO, FIXME,
XXX and HACK — whole-word, ASCII case-insensitive — and prints each hit as
`path:line: text` plus a summary.

## CLI

```sh
granska PATH...
```

```text
$ granska src
src/a.md:2: TODO: fix the parser
src/sub/b.toml:2: # FIXME broken default
2 markers in 2 files
```

Markers must be whole words: `TODOING` does not match, `x-TODO-y` does, and
`todo`/`FixMe` match case-insensitively. When a line contains several markers
the first by kind priority (TODO, FIXME, XXX, HACK) is reported once. Files
that cannot be read as UTF-8 are skipped. Exit: 0 ok, 1 walk error, 2 usage.

## Library

| Item | Meaning |
| --- | --- |
| `Marker { path, line, kind, text }` | One hit: file, 1-based line number, marker kind, trimmed line |
| `marker_in_line(line) -> string` | The matched kind (`"TODO"` …) or `""` |
| `scan_file(path) -> Result<[]Marker, string>` | Scan one file |
| `scan_paths(&[]string) -> Result<[]Marker, string>` | Recursive walk over roots |
| `count_files(&[]Marker) -> i32` | Distinct files for the summary |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.fs` | `list_dir` for the recursive walk — the library declares `capability filesystem` (required by Dunder for directory listing; tests also use `fs.temp_dir`/`remove_all` for fixtures) |
| `sys` | File reads, metadata, stderr |

Run the tests: `dunder test`.
