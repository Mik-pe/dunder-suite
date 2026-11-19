# finn — find text in files

**CLI:** `finn` · **Library:** `import finn` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Finn* is Swedish for "find". A fast, literal text search across directory
trees — a small ripgrep-style tool with no regex engine.

## Purpose

Answer "where in this tree does this text appear?":

- Recursive walk from one or more roots
- Literal substring matching (no regex): predictable, fast, UTF-8 aware
- ASCII case-insensitive mode, extension filter, match cap
- Hidden entries (dot-prefixed) and binary/unreadable files skipped

## CLI (v0.1.0)

```sh
finn "TODO" src/
finn -i "errcode" .                       # ignore ASCII case
finn -l "task scope" .                    # list matching paths only
finn -t ddr,toml "std.net" /path/to/lib   # extension filter
finn --max 5 "parse" .                    # stop after N matches
```

```text
$ finn -t ddr "capability" /Users/mikpe/dev/dunder/stdlib
/Users/mikpe/dev/dunder/stdlib/bytes.ddr:1: capability unsafe
/Users/mikpe/dev/dunder/stdlib/fs.ddr:1: capability filesystem, unsafe
/Users/mikpe/dev/dunder/stdlib/net.ddr:1: capability network, unsafe
```

Output is `path:line: text` (trimmed), sorted by path then line. Exit codes
match grep: 0 matches found, 1 none, 2 error. Paths resolve against the
process working directory — prefer absolute roots in scripts.

## Library

```dunder
import finn

fn main() -> i32 {
    opts := finn.Options {
        pattern: "std.net",
        roots: ["/path/to/lib"],
        extensions: ["ddr"],
        ignore_case: false,
        max_matches: 100,
    }
    match finn.search(&opts) {
        .Ok(matches) => { for m in matches { print(f"{m.path}:{m.line}: {m.text}") } },
        .Err(e) => { print(e) },
    }
    return 0
}
```

| Item | Meaning |
| --- | --- |
| `Options` | Pattern, roots, extension filter, flags, cap |
| `Match { path, line, text }` | One hit |
| `search(&Options) -> Result<[]Match, string>` | Walk, read, scan, sort, cap |
| `collect_files(root, &exts, &mut out, depth)` | Reusable recursive walker |
| `line_matches(line, pattern, ignore_case)` | Case-folding literal scan |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.fs` / `sys` | `list_dir`, `metadata`, `FileKind`, `read_text` |
| `std.text` | `find` for literal scans, `split` lines, `trim` |
| Tasks | `task scope jobs(4)` — files are read on the caller, then line scanning runs in parallel workers |
| `std.array` | `sort_by` with a custom comparator; `append` for merging |

Requires `capability filesystem`.

## v0.1.0 notes

- Dunder's task workers may not perform ambient I/O (transitive effect
  analysis), so v0.1 reads files sequentially on the caller and parallelizes
  the CPU-side scan across four workers — a clean split of I/O and compute.
- The match cap is applied per scan group and again after the merge.

Run the tests: `dunder test` (6 suite tests + stdlib tests).
