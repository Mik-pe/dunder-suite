# titta — poll-based file watcher

**CLI:** `titta` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Titta* is Swedish for "look" — it watches a file by polling its size and
reports the moment it changes.

## CLI

```sh
titta notes.txt                       # poll every 500 ms until size changes
titta --interval-ms 50 log.txt        # faster polling
titta --exec "make build" src/lib.ddr # run a command after the change
```

```text
$ titta --interval-ms 50 watch.txt
changed: watch.txt size 6 -> 11
```

Polling ends after the first size change (`--once` is accepted and is the
default behaviour). The `--exec` command is split on spaces and runs with
inherited stdio.

## Library

| Item | Meaning |
| --- | --- |
| `wait_for_change(path, interval_ms, timeout_ms) -> Result<(i64, i64), string>` | Polls `sys.metadata` every `interval_ms`; returns `(old_size, new_size)`. `timeout_ms <= 0` waits forever; a timeout produces `.Err`. Missing files and intervals < 10 ms are errors |
| `parse_ms(label, text) -> Result<i64, string>` | Shared flag validation (10..3600000) |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `sys` | `metadata` size polling, `sleep_ms`, `now_ms` |
| `std.process` | `Command` + `inherit_stdio` + `exec` for `--exec` — `capability process` (in `src/main.ddr`) |

The tests cover the timeout/error paths and flag validation only; the
live-change path is exercised by the smoke run above.

Run the tests: `dunder test`.
