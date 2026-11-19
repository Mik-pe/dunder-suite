# vilken — PATH executable locator

**CLI:** `vilken` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Vilken* is Swedish for "which" — it locates executables by name along
`$PATH`, printing every regular-file match (deduplicated, in search order).

## CLI

```sh
vilken dunder            # all matches for one name
vilken sh python3        # several names at once
```

```text
$ vilken dunder sh no-such-tool
/Users/mikpe/.dunder/bin/dunder
/Users/mikpe/.local/bin/dunder
/bin/sh
vilken: no-such-tool not found in PATH
(exit status 1)
```

PATH elements are split on `:`, empty elements are skipped, and only regular
files named NAME count (a directory of the same name is ignored). Exit 1 if
any NAME has no match.

## Library

| Item | Meaning |
| --- | --- |
| `search_path(path_value, name) -> []string` | Split `path_value` on `:`, return matching candidate paths in order, deduplicated |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.process` | `env("PATH")` — `capability process` (in `src/main.ddr`) |
| `sys` | `metadata`/`file_kind` regular-file check |

Run the tests: `dunder test`.
