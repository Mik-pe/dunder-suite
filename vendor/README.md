# Vendored Dunder packages

Copied from [`Mik-pe/dunder`](https://github.com/Mik-pe/dunder) `packages/`
at **0.5.0** so this suite builds self-contained (no `path` dependencies that
escape the repo). Package tests were dropped; see the source repo for them.

- `http` — HTTP/1.x server-side request parsing and response writing
- `http_client` — libcurl-backed HTTP client with TLS verification

Used by `hylla` (http) and `salva` (http_client) via their `dunder.toml`:

```toml
[dependencies]
http = { path = "../vendor/http" }
```
