# hylla — static file server

**CLI:** `hylla` · **Library:** `import hylla` · **Status:** planned · **Target:** Dunder 0.5.0

*Hylla* is Swedish for "shelf" — it serves files straight off the shelf.
A small, deliberate HTTP file server on the `http` package and `std.net`.

## Purpose

Share a directory over HTTP without configuration:

- Serve any local directory with correct Content-Type by extension
- Directory index listing with sizes, clickable relative links
- Bounded request bodies for optional uploads (`--upload`, off by default)
- Explicit limits everywhere: max body, header timeouts, path escaping

## CLI

```sh
hylla                          # serve ./ on :8080
hylla --port 9000 --dir dist/  # serve a build directory
hylla --upload /tmp/inbox      # also accept PUT/POST writes under /tmp/inbox
```

Then:

```sh
curl -s http://localhost:8080/            # index listing
curl -sO http://localhost:8080/logo.jpg   # download a file
curl -T report.pdf http://localhost:8080/inbox/report.pdf   # with --upload
```

## Library

```dunder
import hylla

fn main() -> i32 {
    config := hylla.Config {
        root: "dist",
        addr: "0.0.0.0:8080",
        upload_dir: "",        // empty disables uploads
        timeout_ms: 10000,
        max_body: 1048576,
    }
    hylla.serve(&config)       // blocking accept loop
    return 0
}
```

Planned API:

| Item | Meaning |
| --- | --- |
| `Config` | Root directory, bind address, upload dir, timeouts, body limit |
| `serve(&Config)` | Blocking loop: accept → `http.read_request` → route → respond |
| `response_for(&Config, &Request) -> Response` | Pure router, unit-testable without sockets |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `http` package | `read_request`, `write_response`, streaming `BodyReader` for uploads |
| `std.net` | `TcpListener::listen_addr`, `accept`, `local_addr`; sockets are `@resource(dispose)` |
| `sys` / `std.fs` | `metadata` for sizes, path containment checks, upload writes |
| `std.map` | Extension → Content-Type table |

Requires `capability network` and `capability filesystem`.

## Dogfooding value

- Long-running single-binary service: accept loop, socket lifetimes, `@resource` cleanup
- The `http` parser under varied real clients (browsers, curl, pipelines)
- Security-shaped code: path normalization and bounded bodies in a typed language
