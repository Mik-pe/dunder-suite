# vard — host information

**CLI:** `vard` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Värd* is Swedish for "host" — it prints a one-screen summary of the machine
and process the CLI is running on.

## CLI

```sh
vard
```

```text
$ vard
os      macos
arch    arm64
target  arm64-macos
cpus    10
pid     41152
```

Keys align to the longest key plus two spaces.

## Library

| Item | Meaning |
| --- | --- |
| `Pair { key, value }` | One host fact |
| `collect() -> []Pair` | os, arch, target, cpus, pid (live values) |
| `render(&[]Pair) -> string` | Aligned `key  value` lines, joined with `\n` |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.process` | `host_os`, `host_arch`, `host_target`, `host_cpus` — `capability process` |
| `sys` | `pid` |

Run the tests: `dunder test`.
