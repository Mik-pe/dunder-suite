# bas — process supervisor

**CLI:** `bas` · **Library:** `import bas` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Bas* is Swedish for "foreman". Run a set of processes from one file, watch
them, restart what dies with a bounded budget, and stream prefixed output —
a minimal Procfile manager built on `std.process`.

## Purpose

One command to bring a dev stack up and keep it up:

- Reads a `basfile`: `name: program arg arg ...` (# comments, blank lines ok)
- Spawns every entry with piped output; each line is prefixed `[name]`
- Restarts failing entries up to `--restarts N` (default 0); clean exits stay down
- `bas run NAME` runs a single entry in the foreground with inherited stdio
- Returns the worst exit code across all runs

## CLI (v0.1.0)

```sh
bas                                  # run everything in ./basfile
bas --file deploy.basfile            # choose a file
bas --restarts 2                     # allow 2 retries per failing entry
bas run web                          # one entry, foreground
```

```text
$ bas --file basfile --restarts 2
[greet] hello from bas
[bas] greet exited 0
[bas] flaky exited 1
[bas] flaky restarting (attempt 2)
[bas] flaky exited 1
```

## Library

| Item | Meaning |
| --- | --- |
| `Entry { name, program, args }` | One supervisor unit |
| `parse_basfile(content)` / `load(path)` | Parser with line-numbered errors |
| `run_single(&Entry) -> i32` | Foreground exec, returns the exit code |
| `supervise(&[]Entry, restarts) -> i32` | ChildSet loop: `wait_any` poll, `try_wait` drain/reap, restart on nonzero exit |

The supervisor loop is a single-threaded event cycle: `ChildSet.wait_any`
polls the piped fds, `try_wait` drains new output and reaps exits, and line
buffers keep partial lines between rounds. Ctrl-C handling (signal-aware
teardown of the whole set) is planned for v0.2.

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.process` | `Command` (piped stdio, inherit), `Output`, `ChildSet::{new, try_insert, wait_any, try_wait}` with `Result<i32, Child>` |
| `sys` | `decode_u8_range` for output framing, `sleep_ms` restart backoff |

Requires `capability process`. Tests supervise real `/bin/sh` children and
count their runs via marker files — no mocks.

## Dogfooding value

- The `Child`/`ChildSet` resource design under spawn/wait/reap/restart churn
- Byte-stream line framing with `[]u8` buffers and range decodes

Run the tests: `dunder test` (5 suite tests + stdlib tests).
