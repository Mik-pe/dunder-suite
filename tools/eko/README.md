# eko — TCP echo server/client

**CLI:** `eko` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Eko* is Swedish for "echo" — a tiny TCP echo pair: one subcommand serves,
the other sends.

## CLI

```sh
eko serve --port 0 &      # free port, printed to stderr
eko send 127.0.0.1 51009 "hello eko" "second line"
```

```text
$ eko serve --port 0 &        # stderr: eko: serving on 0.0.0.0:51009
$ eko send 127.0.0.1 51009 "hello eko" "second line"
hello eko
second line
```

`serve` handles one connection at a time, echoing every byte back until the
peer disconnects or falls silent for a second. `send` writes each TEXT
argument plus a newline, then prints everything echoed back (HOST must be an
IPv4 dotted quad).

## Library

| Item | Meaning |
| --- | --- |
| `echo_once(&TcpStream) -> i64` | Read→write loop until EOF or a 1000 ms quiet window; returns bytes echoed |
| `roundtrip(port, message) -> Result<string, string>` | Single-threaded self-test: listener + client in one call, echoes `message` and returns the reply |
| `parse_ipv4(host) -> Result<(u8, u8, u8, u8), string>` | Strict dotted-quad parser |
| `bytes_of(s) -> []u8` | String → byte helper |
| `drain_reply(&TcpStream) -> string` | Client-side read-until-quiet used by `send` |

The test suite uses `roundtrip` (listener on port 0, client socket, accept,
`echo_once`, close, read the reply) — the same single-threaded pattern as the
`hylla` tests, no threads or fixtures.

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.net` | `TcpListener::listen_addr`, `accept`, `TcpStream::connect_addr`, `read_append`, `write_all`, `set_nonblocking`, `PollSet`/`poll_in` — `capability network` |
| `sys` | `decode_u8`, `now_ms`, `sleep_ms` |

Run the tests: `dunder test`.
