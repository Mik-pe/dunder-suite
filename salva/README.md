# salva — HTTP load testing

**CLI:** `salva` · **Library:** `import salva` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Salva* is Swedish for a "volley" or "salvo" — a burst of fire. This tool fires
salvos of HTTP requests at a service and reports latency percentiles, built on
the `http_client` package.

## Purpose

Measure a service before you ship it:

- Fixed request count (`-n`) or duration (`-t`) runs
- Latency percentiles (p50/p90/p95/p99), min/max, throughput
- Transport-code classification: connection failures abort with the libcurl code
- Text or JSON report

## CLI (v0.1.0)

```sh
salva -n 200 http://127.0.0.1:8080/health
salva -t 5 http://example.com/                  # duration mode
salva -n 20 --json http://127.0.0.1:8080/ | sil .p95_ms
salva -n 5 -m POST -H content-type=application/json -b '{"x":1}' http://api/items
```

```text
$ salva -n 50 http://127.0.0.1:18433/notes.txt
50 requests in 24 ms (2083.3 rps)
latency ms: p50=0 p90=1 p95=1 p99=6 max=6
transport: failures=0
```

Requests are **sequential** in v0.1: Dunder's task workers may not perform
ambient I/O, so concurrency waits on the runtime. A hard transport failure on
the first request aborts the run (e.g. `transport code 7` = connection
refused). HTTP error statuses count as completed requests (per `http_client`
semantics); a per-status histogram is planned for v0.2.

## Library

| Item | Meaning |
| --- | --- |
| `Config` | URL, method, headers, body, count/duration, timeout, body cap |
| `Sample { status, transport_code, latency_ms, ok, error }` | One completed request |
| `run(&Config) -> Result<([]Sample, wall_ms), string>` | Executes with early abort on first-request failure |
| `summarize(&[]Sample, wall_ms) -> Report` | Percentiles via `sort_by(math.total_cmp_f64)` — pure, unit-tested |
| `render_text` / `render_json` | Report output |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `http_client` package (vendored) | `request(method, url, ...)` with TLS verification |
| `std.math` | `total_cmp_f64` for deterministic float ordering |
| `std.json` | Machine-readable reports |

Requires `capability network` and the system libcurl. Verified live against
`hylla` (2000+ rps on loopback).

Run the tests: `dunder test` (5 suite tests + stdlib tests).
