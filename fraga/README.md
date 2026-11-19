# fraga — SQLite query runner

**CLI:** `fraga` · **Library:** `import fraga` · **Status:** planned · **Target:** Dunder 0.5.0

*Fråga* is Swedish for "query" (the CLI spells it `fraga` for shell-friendliness).
An ad-hoc SQL runner and report formatter over `std.sqlite`.

## Purpose

Turn a `.db` file into an answer in one line:

- Run a single SQL statement with `?` parameters bound safely
- Output as an aligned table (default), CSV, or JSON via `std.toml`-style flags
- Read queries from a file for reusable reports; read-only mode for safety
- Exit status reflects SQL errors, so it composes in shell pipelines

## CLI

```sh
fraga app.db "SELECT id, name FROM users WHERE active = 1"
fraga app.db --csv "SELECT * FROM events LIMIT 100" > events.csv
fraga app.db --json --params "alice,42" "SELECT * FROM users WHERE name = ? OR id = ?"
fraga app.db --file reports/stale-items.sql        # keep SQL in version control
fraga app.db --readonly "PRAGMA integrity_check"
```

Table output:

```text
id  name
--  ------
1   ada
2   grace
(2 rows)
```

## Library

```dunder
import fraga

fn main() -> i32 {
    query := fraga.Query {
        database: "app.db",
        sql: "SELECT id, name FROM users WHERE active = ?",
        params: ["1"],
        read_only: false,
    }
    result := fraga.run(&query)
    if !result.ok { print(result.error); return 1 }
    for row in result.rows { print(row) }   // row: fraga.Row { columns, values }
    return 0
}
```

Planned API:

| Item | Meaning |
| --- | --- |
| `Query` | Database path, SQL, text params, read-only flag |
| `Row { columns: []string, values: []string }` | Decoded from the JSON object strings `std.sqlite` returns; NULL kept as a sentinel |
| `run(&Query) -> Result<Rows, fraga.Error>` | Execute once, format later |
| `render_table(&Rows) -> string` / `render_csv` / `render_json` | Pure formatting helpers, reusable by other tools |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.sqlite` | `Database::open`, `exec`, `query`, `execute_rows` for `RETURNING`; documented limits (1 MiB statement, 1024 params) enforced up front |
| `std.json` | Rows arrive as JSON objects; `Doc` accessors decode values |
| `std.text` / `std.parse` | CSV escaping, param splitting, `--readonly` parsing |

Requires `capability filesystem`. Uses the system SQLite 3 library through the
typed C FFI.

## Dogfooding value

- Heavy `Result<T, E>` discipline: every SQL call is fallible
- Real FFI resource management (`@resource` connection, `defer` close)
- Text shaping: column-width alignment with `f"{w}"`-style padding and rune-aware truncation
