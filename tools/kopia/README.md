# kopia — timestamped backups

**CLI:** `kopia` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Kopia* is Swedish for "copy" — it makes byte-exact backups of a file under
timestamped sibling names and prunes the old ones.

## CLI

```sh
kopia doc.txt            # writes doc.txt.bak.<now-ms>, keeps 5 newest
kopia --keep 2 doc.txt   # keep only the 2 newest .bak.<ms> siblings
```

```text
$ kopia --keep 2 doc.txt
doc.txt.bak.1795124715865
pruned 1 old backup(s)
```

Pruning considers only siblings named `<base>.bak.<milliseconds>` with a
parsable trailing number; anything else is left alone.

## Library

| Item | Meaning |
| --- | --- |
| `backup(path) -> Result<string, string>` | Copy `path` bytes to `path.bak.<now_ms>`; returns the new path |
| `prune(dir, base_name, keep) -> Result<i32, string>` | Remove the oldest `.bak.<ms>` siblings beyond `keep`; returns how many were removed |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.fs` | `list_dir` for pruning — `capability filesystem` |
| `sys` | `read_u8`/`write_u8`, `remove_file`, `now_ms` |
| `std.parse` | Trailing-timestamp parsing |

Run the tests: `dunder test`.
