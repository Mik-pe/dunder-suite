# dublett — duplicate file finder

**CLI:** `dublett` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Dublett* is Swedish for "duplicate" — it finds byte-identical files under
one or more paths by grouping on size first and confirming with sha256.

## CLI

```sh
dublett .                  # scan current tree
dublett dir1 dir2          # scan several roots
```

```text
$ dublett photos/
bbbff613b27d7f717f8e7bfce522186e0e2539efd86520ceac00ce62dab458c2  5  photos/a.txt
bbbff613b27d7f717f8e7bfce522186e0e2539efd86520ceac00ce62dab458c2  5  photos/b.txt
1 groups, 2 duplicate files
```

Rows are `hash  size  path`, sorted by hash then path; the summary counts
groups and files participating in them. Hidden entries are skipped and
unreadable files are ignored.

## Library

| Item | Meaning |
| --- | --- |
| `FileEntry { path, size, digest }` | One regular file; `digest` may be `""` until hashed |
| `file_digest(path) -> Result<string, string>` | Hex sha256 of a file's bytes |
| `collect_entries(root) -> Result<[]FileEntry, string>` | Recursive walk (hidden skipped, regular files only) |
| `size_candidates(&[]FileEntry) -> []FileEntry` | Pure filter: keep entries whose size repeats |
| `group_by_digest(&[]FileEntry) -> [][]string` | Pure grouping: path groups (2+) sorted by digest, then path |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.fs` | `list_dir` for the walk — `capability filesystem` |
| `std.sha256`, `std.bytes` | Digest + hex rendering (digest works on `[]i32`, so bytes are converted both ways) |
| `sys` | `read_u8`, `metadata`/`file_kind` |

Run the tests: `dunder test`.
