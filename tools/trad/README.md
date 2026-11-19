# trad — directory tree

**CLI:** `trad` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Trad* is Swedish for "thread" — it draws a directory tree as a thread of
`|-- ` / `` `-- `` connectors, depth-capped and sorted per directory.

## CLI

```sh
trad                 # tree of . up to depth 4
trad -d 2 src/       # shallower tree of src/
trad -d 0 FILE       # prints the path as-is (files render bare)
```

```text
$ trad -d 2 tests/
tests/
|-- build/
|   `-- arm64-macos/
|-- dunder.toml
|-- src/
|   |-- main.ddr
|   `-- trad.ddr
`-- tests/
    `-- suite.ddr
```

Hidden (dot-prefixed) entries are skipped, directories get a trailing `/`,
and unreadable subdirectories render as bare directories.

## Library

| Item | Meaning |
| --- | --- |
| `render_tree(root, max_depth) -> Result<string, string>` | Recursive tree rendering; `.Err` for unreadable root or negative depth. Depth counts levels below `root` (0 = root only) |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.fs` | `list_dir` (sorted entries) — `capability filesystem` |
| `sys` | `metadata`/`file_kind` for the `/` suffix |

Run the tests: `dunder test`.
