# fraga — SQLite query runner

**CLI:** `fraga` · **Library:** `import fraga` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Fråga* is Swedish for "query" (the CLI spells it `fraga` for shell-friendliness).
An ad-hoc SQL runner and report formatter over `std.sqlite`.

## Purpose

Turn a `.db` file into an answer in one line:

- Run a single SQL statement with `?` parameters bound safely
- Output as an aligned table (default), CSV with quoting, or JSON
- Read queries from a file; read-only mode for safety
- Exit status reflects SQL errors, so it composes in shell pipelines

## CLI (v0.1.0)

```sh
fraga app.db "SELECT id, name FROM users WHERE active = 1"
fraga app.db --csv "SELECT * FROM events LIMIT 100" > events.csv
fraga app.db --json --params "alice,42" "SELECT * FROM users WHERE name = ? OR id = ?"
fraga app.db --file reports/stale-items.sql
fraga app.db --readonly "PRAGMA integrity_check"
```

```text
$ fraga /tmp/fraga-demo.db "SELECT id, name FROM items"
id  name
--  -----------
1   hello
2   with, comma
3   say "hi"
(3 rows)
```

Reads (`SELECT`/`PRAGMA`/`WITH`/`EXPLAIN` by leading word) render rows; other
statements execute and report `(N changes)`. NULL renders as an empty cell;
`--json` emits values as strings (pipe through `sil` for typed reshaping).

## Library

```dunder
import fraga

fn main() -> i32 {
    query := fraga.Query {
        database: "app.db",
        sql: "SELECT id FROM users WHERE active = ?",
        params: ["1"],
        read_only: true,
    }
    match fraga.run(&query) {
        .Ok(rows) => { print(fraga.render_table(&rows)) },
        .Err(e) => { print(e) },
    }
    return 0
}
```

| Item | Meaning |
| --- | --- |
| `Query` | Database path, SQL, text params, read-only flag |
| `run` / `run_any` / `execute` | Read-only query, any statement with rows, write with change count |
| `Rows { columns, rows }` | Decoded columns and text values |
| `render_table` / `render_csv` / `render_json` | Pure formatting helpers |
| `looks_like_read(sql)` | Leading-word statement classification |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.sqlite` | `Database::open`, `exec`, `query`, `execute_rows`; single-statement and parameter limits enforced by the module |
| `std.json` | Rows arrive as JSON objects; `Doc` decodes them |
| `std.text` / `std.bytes` | CSV escaping, column alignment |

Requires `capability filesystem` and the system SQLite 3 library.

## Dogfooding value

- Heavy `Result<T, E>` discipline: every SQL call is fallible
- `@resource` connection with `defer` close under real churn
- Text shaping: width alignment and byte-exact CSV quoting

Run the tests: `dunder test` (7 suite tests + stdlib tests).
