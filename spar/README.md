# spar — structured logging

**Library:** `import spar` · **Viewer CLI:** `spar view` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Spar* is Swedish for "trace" (the CLI spells it `spar` for shell-friendliness).
A structured logging library that emits JSON Lines, plus a viewer CLI to read
them back.

## Purpose

Give every Dunder program logs that machines and humans can both consume:

- Library: levelled, field-carrying JSONL events to stderr or an append-only file
- Viewer: filter by level, field equality, message substring, and tail count
- No global mutable state: a `Logger` is an owned value you pass explicitly
- One JSON schema end to end: what the library writes is what the viewer reads

## Library (v0.1.0)

```dunder
import spar

fn main() -> i32 {
    logger := spar.Logger::new("hylla").with_file("app.log")
    logger.info("listening", &[spar.field_i64("port", 8080)])
    logger.warn("slow request", &[spar.field_f64("ms", 1420.5)])
    logger.error("upload rejected", &[spar.field_str("path", "../secrets")])
    return 0
}
```

Each call appends one line (`ts` from `sys.now_ms()`, wall clock):

```json
{"ts":1763640000,"mod":"hylla","level":"info","msg":"listening","port":8080}
{"ts":1763640009,"mod":"hylla","level":"warn","msg":"slow request","ms":1420.5}
```

| Item | Meaning |
| --- | --- |
| `Logger::new(module)` | Owned logger writing to stderr |
| `.with_level(Level)` / `.with_file(path)` | Consuming builders; file mode appends |
| `.debug/.info/.warn/.error(msg, &[]Field)` | One JSONL event each |
| `field_str` / `field_i64` / `field_f64` / `field_bool` | Typed fields |
| `parse_level(text)` / `level_value(Level)` | Level parsing and ordering |
| `view(input, &Query) -> []string` | The viewer's filter + pretty renderer |
| `value_text(&Doc, node)` | JSON scalar → comparable text |

## Viewer CLI (v0.1.0)

```sh
spar view app.log                     # pretty-print every line
spar view --level warn app.log        # warn and above
spar view --field mod=hylla app.log   # field equality
spar view --has "slow" app.log        # msg substring
spar view --tail 20 -                 # stdin, last 20 matching
```

```text
1763640000 INFO  hylla listening port=8080
1763640009 WARN  hylla slow request ms=1420.5
1763640010 ERROR lod probe failed host=db
```

Non-JSON lines are dropped. `-` or no file reads stdin via `/dev/stdin`.
`SPAR_LEVEL`/`SPAR_FILE` env support is planned for v0.2.

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.json` | `Writer` for events, `Doc` parsing in the viewer |
| `sys` | `now_ms`, `eprint`, `append_text` |
| `std.text` | `split`, `trim`, `find` |

File output needs `capability filesystem`; stderr-only logging needs none.

## Dogfooding value

- A library-first design where the CLI is a thin consumer
- `std.json.Writer` under sustained, interleaved event traffic
- Self-hosting finale: sibling tools log through `spar`, and `spar view` reads
  them back — the suite watches itself

Run the tests: `dunder test` (6 suite tests + stdlib tests).
