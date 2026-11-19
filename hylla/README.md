# hylla — static file server

**CLI:** `hylla` · **Library:** `import hylla` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Hylla* is Swedish for "shelf" — it serves files straight off the shelf.
A small, deliberate HTTP file server on the `http` package and `std.net`.

## Purpose

Share a directory over HTTP without configuration:

- Serve any local directory with correct Content-Type by extension
- `index.html` for `/`; HTML directory listing (HTML-escaped) otherwise
- Bounded uploads (`--upload`, off by default), stored under their basename
- Explicit limits: dot-dot segments rejected, request timeout, body cap

## CLI (v0.1.0)

```sh
hylla                            # serve ./ on :8080
hylla --port 9000 --dir dist/    # serve a build directory
hylla --port 0 --upload inbox/   # free port, accept PUT/POST under inbox/
```

```text
$ curl -s http://localhost:8080/notes.txt
hello from hylla
$ echo "upload test" | curl -s -T - http://localhost:8080/inbox.txt
stored
```

Each request is handled on its own connection (Connection: close). HEAD gets
correct bodiless responses. Traversal attempts (`/../`) return 404.

## Library

| Item | Meaning |
| --- | --- |
| `Config` | Root, port, upload dir, timeout, body cap |
| `route(root, upload, method, target)` | Pure router: `File`/`Listing`/`Upload`/`NotFound`/`NotAllowed` — unit-tested without sockets |
| `handle(&stream, &Config)` | One request/response cycle; used by `serve` and by the loopback test |
| `serve(&Config) -> i32` | Blocking accept loop |
| `content_type(path)` | Extension → MIME mapping |

The test suite drives a real request over loopback with a `TcpListener` and a
client `TcpStream` in one thread — no fixtures, no network.

## Dunder capabilities used

| Area | What |
| --- | --- |
| `http` package (vendored) | `read_request`, `write_response` |
| `std.net` | `TcpListener::listen_addr`, `accept`, `read_append`, `write_all`; `@resource` socket cleanup |
| `sys` | `metadata`/`FileKind`, `read_u8`/`write_u8` for files and uploads |

Requires `capability network` and `capability filesystem`. The `http` package
is vendored under `vendor/http` (copied from Mik-pe/dunder at 0.5.0) so this
repo builds self-contained.

## Dogfooding value

- Long-running accept loop with per-connection resource lifetimes
- The `http` parser under real clients (curl, salva, browsers)

Run the tests: `dunder test` (5 suite tests + stdlib tests).
