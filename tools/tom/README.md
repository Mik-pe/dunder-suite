# tom — stale-file cleaner

**CLI:** `tom` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Tom* is Swedish for "empty" — it empties a directory of stale leftover
files, selected by filename marker.

## Simplification (documented)

Dunder's `sys.metadata` exposes only size and kind — no modification times —
so "age" cannot be computed. `tom` therefore uses a **marker convention**:
a file is stale when its *name ends* with the marker suffix.

## CLI

```sh
tom build/                  # list what would go
tom --marker .old --run db/ # really delete *.old files
```

```text
$ tom --run build/
removed build/one.tmp
removed build/two.tmp
2 file(s) removed
```

Non-recursive; directories whose name ends in the marker are ignored; only
regular files match.

## Library

| Item | Meaning |
| --- | --- |
| `candidates(dir, suffix) -> []string` | Sorted full paths of regular files in `dir` ending with `suffix` |
| `remove_all_files(&[]string) -> Result<i32, string>` | Remove each path; returns the count removed. `.Err` when there were paths but none could be removed |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.fs` | `list_dir` — `capability filesystem` |
| `sys` | `metadata`/`file_kind`, `remove_file` |

Run the tests: `dunder test`.
