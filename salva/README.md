# salva — HTTP load testing

**CLI:** `salva` · **Library:** `import salva` · **Status:** planned · **Target:** Dunder 0.5.0

*Salva* is Swedish for a "volley" or "salvo" — a burst of fire. This tool fires
salvos of HTTP requests at a service and reports latency percentiles, built on
the `http_client` package.

## Purpose

Measure a service before you ship it:

- Fixed request count (`-n`) or duration (`-t`) runs
- Concurrency via isolated tasks (`-c`), each task issuing sequential requests
- Latency percentiles (p50/p90/p95/p99), throughput, status-code histogram
- Transport-code classification (connect failure vs timeout vs HTTP error)
- JSON or plain-text report; histogram feeds straight into `rita hist`

## CLI

```sh
salva -n 200 -c 8 https://api.example.com/health
salva -n 1000 -c 16 -m POST -b '{"x":1}' -H content-type=application/json https://api.example.com/items
salva -t 30s -c 4 --json https://example.com/ | rita hist --field latency_ms
```

Sample text report:

```text
200 requests in 3.9s (51.3 rps)
latency ms: p50=18 p90=42 p95=61 p99=140 max=210
status: 200=196 404=3 500=1
transport: timeouts=0 refused=0
```

## Library

```dunder
import salva

fn main() -> i32 {
    config := salva.Config {
        url: "https://api.example.com/health",
        total: 200,
        concurrency: 8,
        method: "GET",
        timeout_ms: 5000,
    }
    report := salva.run(&config)
    if !report.ok { print(report.error); return 1 }
    print(f"p95 = {report.p95} ms over {report.count} requests")
    return 0
}
```

Planned API:

| Item | Meaning |
| --- | --- |
| `Config` | URL, method, headers, body, total/duration, concurrency, timeout |
| `Sample { status, transport_code, latency_ms }` | One completed request |
| `Report` | Count, rps, sorted latencies, percentiles, `Map<i32, i64>` status histogram |
| `run(&Config) -> Report` | Executes via `task scope`, then reduces |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `http_client` package | `request(method, url, ...)`; `transport_code` drives error classes |
| Tasks | `task scope pool(c)`; each worker accumulates `[]Sample` (`Sendable`), merged with `await!` |
| `std.math` / `std.array` | Percentiles via `sort_by(total_cmp_f64)` on the latency array |
| `std.json` | Machine-readable report output |

Requires `capability network` and the `http_client` package. Concurrent tasks
need a libcurl with thread-safe global init (documented `http_client` caveat);
`-c 1` gives a deterministic sequential mode otherwise.

## Dogfooding value

- Sustained mixed I/O + timing workload (`sys` monotonic clock)
- Large `[]f64` sorting and statistics on real distributions
- Integration between packages: `http_client` + tasks + JSON writer
