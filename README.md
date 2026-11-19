# Dunder Suite

Ten CLIs and libraries written in [Dunder](../dunder), usable for different
purposes and doubling as real-world test programs for the language, compiler
and standard library — the same role Hundmat plays for graphics and servers.

**Status:** all ten tools implemented at **v0.1.0** with passing tests
(232 suite tests green against the installed 0.5.0 compiler). Implementation
feedback lives in [DUNDER-ISSUES.md](DUNDER-ISSUES.md).

## Catalogs

| # | Catalog | CLI | Status | Purpose | Core Dunder |
| --- | --- | --- | --- | --- | --- |
| 1 | [finn](finn/) | `finn` | **v0.1.0** | Recursive literal text search across file trees | `std.text`, `std.fs`, tasks |
| 2 | [salva](salva/) | `salva` | **v0.1.0** | HTTP load testing with latency percentiles | `http_client`, `std.math` |
| 3 | [hylla](hylla/) | `hylla` | **v0.1.0** | Static file server with directory index and uploads | `http`, `std.net`, `std.fs` |
| 4 | [valv](valv/) | `valv` | **v0.1.0** | Passphrase-encrypted local secrets vault | `std.crypto`, `std.json` |
| 5 | [fraga](fraga/) | `fraga` | **v0.1.0** | Ad-hoc SQLite query runner with table/CSV/JSON output | `std.sqlite`, `std.json` |
| 6 | [sil](sil/) | `sil` | **v0.1.0** | jq-lite path selection over JSON, JSONL and TOML | `std.json`, `std.toml` |
| 7 | [bas](bas/) | `bas` | **v0.1.0** | Procfile-style process supervisor with restarts | `std.process`, `sys` |
| 8 | [spar](spar/) | `spar` | **v0.1.0** | Structured JSONL logging library and log viewer | `std.json`, `sys` |
| 9 | [lod](lod/) | `lod` | **v0.1.0** | TCP port probes and scans with CI-friendly exit codes | `std.net`, `std.json` |
| 10 | [rita](rita/) | `rita` | **v0.1.0** | ASCII bar charts, histograms and sparklines | `std.parse`, `f64`, runes |

The `http` and `http_client` packages are vendored under [vendor/](vendor/) so
this repo builds self-contained: `git clone` + `dunder test` just works.

## How the pieces fit together

```text
                 ┌─────────────────────────────────────────┐
                 │                bas (supervisor)         │
                 |  web: hylla   api: ./server   jobs: …   │
                 └───────┬──────────────────┬──────────────┘
                         │ serves           │ watches
                         ▼                  ▼
   salva ──load──▶ hylla ──serves──▶ rita (wasm) rendered charts
      │                                        ▲
      │ --json                                 │ hist --field latency_ms
      ▼                                        │
    sil .p95 ────── all tools log through ──▶ spar ──▶ spar view --level warn
                         │
                         ▼
              fraga (results in SQLite) ──▶ lod probes everything
```

- Every tool emits structured logs through `spar`
- Every JSON stream is filterable by `sil` and plottable by `rita`
- `lod` health-checks services; `bas` keeps them running; `finn` searches them all

## Conventions

- One catalog = one project: `dunder.toml` with `name`, `root` (`src/main.ddr`),
  `test_root` (`tests/suite.ddr`); the library lives beside the CLI as
  `src/<name>.ddr` and is imported by tests via `import ..src.<name>`
- Each v0.1.0 tool runs with `dunder run src/main.ddr -- <args>` and tests
  with `dunder test` from its catalog directory
- Capabilities (`network`, `filesystem`, `process`) are declared explicitly and
  minimally per tool — `sil` and `rita` need none
- Tests use `@test fn` runners; fixtures are synthetic (temp dirs, loopback
  listeners) per dunder's AGENTS.md
- Native targets are ARM64/macOS, ARM64/Linux, x86-64/Linux; `sil` and `rita`
  aim to also build for wasm32 since they are pure computation
- Argument parsing is stdlib-only today (`sys.arg`); if the tools converge on
  a shared flags helper, it may be extracted into an eleventh shared library

## Versioning

The suite tracks Dunder releases. v0.5.0 alignment notes:

- HTTP server/client come from the `packages/http` and `packages/http_client`
  Dunder packages (the `std.http` transition imports are gone in 0.5.0)
- Tasks, `ChildSet`, `@resource(dispose)` sockets and conditional trait impls
  are all stable on latest main and assumed by these plans
- Tools must keep building with `dunder check --locked` against a pinned
  `dunder.lock` once implementation starts
