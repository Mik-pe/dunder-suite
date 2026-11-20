# kolla — sha256 checksum tool

**CLI:** `kolla` · **Library:** `import kolla` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Kolla* is Swedish for "check/look". `kolla` prints sha256 checksums in the
familiar `<hex>  <path>` format and verifies such lists, matching `shasum -a
256` output exactly (lowercase hex, two-space separator).

## CLI

```sh
kolla FILE...                 # print checksums
kolla --check LIST            # verify lines of "<hex>  <path>"
```

```text
$ printf 'hello\n' > a.txt
$ kolla a.txt
5891b5b522d5df086d0ff0b110fbd9d21bb4fc7163af34d08286a2e846f6be03  a.txt
$ kolla a.txt > a.txt.sha256 && kolla --check a.txt.sha256
a.txt: OK
```

`--check` prints one `path: OK` / `path: FAIL (...)` line per entry and exits
1 on any mismatch, unreadable file, or malformed line. Files are capped at
64 MiB.

## Library

| Item | Meaning |
| --- | --- |
| `checksum(path: string) -> Result<string, string>` | Whole-file sha256 as lowercase hex; errors on unreadable path, directory, or >64 MiB |
| `verify_line(line: string) -> Result<string, string>` | Verify one `"<hex>  <path>"` line; `Ok("<path>: OK")` or `Err("<path>: FAIL (...)")`; tolerates a trailing newline |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.sha256`, `std.bytes` | `digest_bytes` + hex rendering |
| `sys` | metadata / raw file reads — no capability line required |
| `std.fs` (tests only) | `capability filesystem` for temp-dir fixtures |

Run the tests: `dunder test`.
