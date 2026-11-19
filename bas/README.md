# bas — process supervisor

**CLI:** `bas` · **Library:** `import bas` · **Status:** planned · **Target:** Dunder 0.5.0

*Bas* is Swedish for "foreman". Run a set of processes from one file, watch
them, restart what dies, and stream prefixed output — a minimal Procfile
manager built on `std.process`.

## Purpose

One command to bring a dev stack up and keep it up:

- Reads a `basfile` (or `Procfile`): `name: command arg arg ...`
- Spawns every entry, prefixes each line of output with its name
- Restarts crashed processes with backoff; `bas run <name>` for a single one
- Ctrl-C tears the whole set down via `terminate` on remaining children

## CLI

```sh
bas                       # run everything in ./basfile
bas -f deploy.basfile     # choose a file
bas run worker            # just one entry, foreground-style
bas --no-restart flaky    # let it die and report the exit code
```

A `basfile` for a small web stack:

```text
web:     hylla --port 8080 --dir dist
api:     ./server --listen 127.0.0.1:9000
worker:  ./jobs --queue default
```

Output:

```text
[web]  serving dist on 0.0.0.0:8080
[api]  listening on 127.0.0.1:9000
[worker] exited 1, restarting in 2s (attempt 2)
```

## Library

```dunder
import bas

fn main() -> i32 {
    mut spec := bas.Spec::new()
    spec.add("web", "hylla", ["--port", "8080"])
    spec.add("worker", "./jobs", [])
    supervisor := bas.Supervisor::new(&spec)
    return supervisor.run()      // blocks until stopped; returns worst exit code
}
```

Planned API:

| Item | Meaning |
| --- | --- |
| `Spec` | Ordered entries: name, program, args, restart policy |
| `load(path) -> Result<Spec, Error>` | Parses a basfile/Procfile |
| `Supervisor::run()` | Spawn loop over `ChildSet`; readers via tasks; restart backoff |
| `Event { kind, name, detail }` | Structured lifecycle events, feedable into `spar` |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.process` | `Command::spawn`, `Child.wait`/`try_wait`/`read_stdout`/`terminate`, `ChildSet::try_insert` with `Result<i32, Child>` |
| Tasks | One reader task per child merging prefixed lines; `Sendable` event batches |
| `sys` | Clock for backoff timing, `sleep_ms` |

Requires `capability process`. On Linux, `std.process` uses direct syscalls —
the supervisor stays a static ELF.

## Dogfooding value

- The `ChildSet`/`Child` resource design under continuous churn: spawn, wait, reap, restart
- Task-to-task coordination with owned reads (stdout streams never copied twice)
- Supervising real suite binaries (`hylla`, project servers) as its own test fixture
