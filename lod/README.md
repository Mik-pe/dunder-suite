# lod — network probe

**CLI:** `lod` · **Library:** `import lod` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Lod* is Swedish for "sounding line" — the weight you drop to find the bottom.
A TCP port prober with port scans, health checks and CI-friendly exit codes.

## Purpose

Answer "is this thing reachable?" across a fleet of targets:

- `probe`: TCP connect checks for `host:port` targets with a deadline each
- `scan`: sweep a port range on one host (`80,443,8000-8004`)
- Table or JSON output; exit code reflects failures

## CLI (v0.1.0)

```sh
lod probe db.internal:5432 cache:6379 '[::1]:443'
lod scan 127.0.0.1 8000-8004,9000 --timeout-ms 500
lod probe --json api:443 | sil .  # machine output composes with sil
```

```text
$ lod probe localhost:80 127.0.0.1:1 example.invalid:80
host              port  state  latency_ms
localhost           80  closed  -
127.0.0.1            1  closed  -
example.invalid     80  closed  -
```

Targets accept `host:port`, `[v6]:port`. Exit: 0 all open, 1 some closed,
2 error (bad target, bad port spec).

## Library

```dunder
import lod

fn main() -> i32 {
    match lod.parse_target("db.internal:5432") {
        .Ok(pair) => {
            result := lod.probe(pair.0, pair.1, 1000)
            print(f"open={result.open} in {result.latency_ms} ms")
        },
        .Err(e) => { print(e) },
    }
    return 0
}
```

| Item | Meaning |
| --- | --- |
| `Probe { host, port, open, latency_ms, error }` | One connect outcome |
| `parse_target(text) -> Result<(string, u16), string>` | `host:port` / `[v6]:port` |
| `probe(host, port, timeout_ms)` | `connect_host` + monotonic timing; closes the stream |
| `probe_targets(&[]string, timeout)` / `probe_ports(host, &[]i32, timeout)` | Batches |
| `parse_ports(spec) -> Result<[]i32, string>` | Comma lists and `a-b` ranges |
| `count_closed(&[]Probe) -> i32` | For exit codes |
| `render_table` / `render_json` | Output helpers |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.net` | `TcpStream::connect_host` (DNS + connect deadline), `@resource` socket cleanup |
| `std.json` | `Writer` for JSON output |
| `std.parse` / `std.text` | Port specs, targets |

Requires `capability network`.

## v0.1.0 notes

- Probes run sequentially, each with its own deadline: Dunder's task workers
  may not perform ambient I/O (transitive effect analysis), so parallel
  probing waits for ambient-I/O-capable workers or a spawn-per-probe process
  model.
- The suite's own tests open a real `TcpListener` on loopback and probe it —
  no external fixtures.

Run the tests: `dunder test` (5 suite tests + stdlib tests).
