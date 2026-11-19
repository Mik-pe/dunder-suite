# sil — structured data filter

**CLI:** `sil` · **Library:** `import sil` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Sil* is Swedish for "sieve". A jq-lite for JSON, JSON Lines and TOML:
select paths and convert TOML to JSON.

## Purpose

Filter and reshape structured data without a full query language:

- Path selection like `.user.name` and `.items[2].id` into scalars or subtrees
- JSONL streaming mode: apply the same path per line (`--lines`)
- Convert TOML → JSON (`--toml`), covering `dunder.toml`
- Pretty-print with stable key order; `--raw` for unquoted scalars

## CLI (v0.1.0)

```sh
sil .name user.json                  # select a scalar (strings print quoted)
sil -r .name user.json               # -r: print string values unquoted
sil -p . user.json                   # -p: pretty-print the selection
sil .dependencies.http.path ../dunder/dunder.toml --toml
sil --lines .level logs.jsonl        # one value per JSONL line
echo '{"a":[1,2,3]}' | sil .a        # no FILE (or '-') reads stdin
```

The path grammar: `.key`, chained with `.` or attached `[N]`, e.g.
`.items[2].id`, `.a.b[0]`, and `.` alone for the whole document. Errors name
the failing segment (`no key "x" at .user`).

In `--lines` mode the first malformed line is reported with its line number
and the tool exits 1. `--set` edits and array slices are planned for v0.2.

## Library

```dunder
import sil

fn main() -> i32 {
    match sil.parse_path(".user.name") {
        .Ok(segments) => {
            doc := json.Doc::parse_text("{\"user\": {\"name\": \"ada\"}}")
            picked := sil.select(&doc, doc.root, &segments)
            if picked.ok { print(doc.as_str(picked.node).value) }
        },
        .Err(e) => { print(e) },
    }
    return 0
}
```

| Item | Meaning |
| --- | --- |
| `Segment` | Enum: `Key(string)` or `Index(i32)` |
| `parse_path(text) -> Result<[]Segment, string>` | Grammar above; rejects empty/ambiguous paths |
| `select(&Doc, from, &[]Segment) -> Selection` | Walks `field`/`at`; typed error names the segment |
| `render_pretty(&Doc, node) -> string` | Two-space indented JSON |
| `toml_to_json(text) -> Result<string, string>` | `std.toml` bridge with its parse errors |
| `describe(&[]Segment) -> string` | Canonical path text |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.json` | `Doc::parse_text`, `field`/`at`/`kind`/`as_str`, `Writer` |
| `std.toml` | `parse` → JSON rendering |
| `std.parse` / `std.text` | Path lexing, integer segments |

No capabilities required — pure computation, streams stdin → stdout (via
`/dev/stdin`, since `sys` has no explicit stdin reader yet).

## Dogfooding value

- Parser + enum-payload state machine with precise error positions
- A daily-driver tool other suite members call (`spar view --json | sil .msg`)
- Candidate first wasm32 build in the suite

Run the tests: `dunder test` (10 suite tests + stdlib tests).
