# Dunder Suite

Ten CLIs and libraries written in [Dunder](../dunder), usable for different
purposes and doubling as real-world test programs for the language, compiler
and standard library — the same role Hundmat plays for graphics and servers.

**Status:** ten primary tools at **v0.1.0** (232 tests) plus **fifty
utilities** under `tools/` (693 tests) — all green against the installed
Dunder 0.5.0 compiler (925 tests total). Implementation feedback lives in
[DUNDER-ISSUES.md](DUNDER-ISSUES.md); conventions in [AGENT-GUIDE.md](AGENT-GUIDE.md).

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

## The fifty — `tools/`

Fifty smaller utilities, each with the same layout and testing bar as the ten
above. Built in five parallel batches (text, math, encoding/crypto, data
formats, files/system); **693 tests, 0 failures** across all fifty.

### Text (batch T)

| Tool | Purpose | Tool | Purpose |
| --- | --- | --- | --- |
| [ord](tools/ord/) | wc-style line/word/byte counts | [slug](tools/slug/) | slugify with unicode stripping |
| [klipp](tools/klipp/) | cut-style field extractor | [versal](tools/versal/) | ASCII upper/lower/title case |
| [unik](tools/unik/) | collapse consecutive duplicates | [radbryt](tools/radbryt/) | word wrap to a width |
| [vand](tools/vand/) | reverse lines or runes | [numrera](tools/numrera/) | number lines (nl-style) |
| [fyll](tools/fyll/) | pad/center/truncate lines | [granska](tools/granska/) | TODO/FIXME marker scanner |

### Math (batch M)

| Tool | Purpose | Tool | Purpose |
| --- | --- | --- | --- |
| [rakna](tools/rakna/) | integer expression evaluator | [slump](tools/slump/) | random hex/int/pick |
| [bas](tools/bas/) | base converter (2/8/10/16) | [hexdump](tools/hexdump/) | offset+hex+ascii dump |
| [prim](tools/prim/) | prime check/factor/nth/list | [procent](tools/procent/) | percentage math |
| [stats](tools/stats/) | mean/median/stdev summaries | [tarning](tools/taring/) | dice notation roller |
| [fibo](tools/fibo/) | fibonacci list/nth | [omvandla](tools/omvandla/) | unit converter |

### Encoding & crypto (batch E)

| Tool | Purpose | Tool | Purpose |
| --- | --- | --- | --- |
| [rot13](tools/rot13/) | Caesar/rot-N cipher | [gom](tools/gom/) | AES-256-GCM file encryption |
| [b64](tools/b64/) | base64 encode/decode | [slumpord](tools/slumpord/) | passphrase generator |
| [urlkod](tools/urlkod/) | percent-encoding | [skillnad](tools/skillnad/) | line diff (LCS) |
| [kolla](tools/kolla/) | sha256 checksums + verify | [citera](tools/citera/) | JSON string quote/unquote |
| [stempel](tools/stempel/) | HMAC-SHA256 sign/verify | [tidsstampel](tools/tidsstampel/) | unix timestamp ↔ civil time |

### Data formats (batch D)

| Tool | Purpose | Tool | Purpose |
| --- | --- | --- | --- |
| [json2csv](tools/json2csv/) | JSON array → CSV | [lagra](tools/lagra/) | key-value store on JSON |
| [csv2json](tools/csv2json/) | CSV → JSON (quoted cells) | [param](tools/param/) | query string ↔ JSON |
| [toml2json](tools/toml2json/) | TOML → pretty JSON | [kolonn](tools/kolonn/) | aligned table from TSV |
| [packa](tools/packa/) | JSON minifier | [sortera](tools/sortera/) | sort lines (numeric/unique) |
| [validera](tools/validera/) | JSON validator with offsets | [snygga](tools/snygga/) | pretty-printer with options |

### Files & system (batch S)

| Tool | Purpose | Tool | Purpose |
| --- | --- | --- | --- |
| [trad](tools/trad/) | directory tree | [titta](tools/titta/) | poll watcher + exec |
| [dublett](tools/dublett/) | duplicate file finder (sha256) | [kopia](tools/kopia/) | timestamped backups |
| [storlek](tools/storlek/) | recursive directory sizes | [byt](tools/byt/) | batch rename substring |
| [vilken](tools/vilken/) | PATH executable locator | [tom](tools/tom/) | suffix-based file cleaner |
| [vard](tools/vard/) | host info report | [eko](tools/eko/) | TCP echo server/client |

`AGENT-GUIDE.md` documents the Dunder conventions used across all sixty
projects — including the gotchas that shaped them.

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
