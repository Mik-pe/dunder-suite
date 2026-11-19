# tidsstampel — unix timestamp converter

**CLI:** `tidsstampel` · **Library:** `import tidsstampel` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Tidsstampel* is Swedish for "time stamp". It converts between unix seconds
and `YYYY-MM-DD HH:MM:SS` civil dates using Howard Hinnant's public-domain
`days_from_civil` / `civil_from_days` algorithms. All math is `i64` seconds,
**UTC only** — there is no local-time handling and no leap seconds.

## CLI

```sh
tidsstampel NU                       # current unix time (seconds)
tidsstampel T 1763640000             # -> 2025-11-20 12:00:00 UTC
tidsstampel T -1                     # -> 1969-12-31 23:59:59 UTC
tidsstampel D "2025-11-20 12:00:00"  # -> 1763640000
tidsstampel D "2023-02-29 00:00:00"  # -> error, exit 1
```

Supported range is years 1..9999 (`0001-01-01 00:00:00` = -62135596800
through `9999-12-31 23:59:59` = 253402300799). Parsing is strict: 19
characters, digits only, `-`/space/`:` separators, real month/day ranges
(Feb 29 only in leap years), and the date must exist.

## Library

| Item | Meaning |
| --- | --- |
| `days_from_civil(y: i64, m: i32, d: i32) -> i64` | Hinnant: days since 1970-01-01 |
| `civil_from_days(days: i64) -> (i64, i32, i32)` | Hinnant inverse: `(y, m, d)` |
| `format_timestamp(sec: i64) -> Result<string, string>` | `"YYYY-MM-DD HH:MM:SS UTC"`; `Err` outside years 1..9999; floor-division handles negative seconds |
| `parse_timestamp(s: string) -> Result<i64, string>` | Strict parse of `"YYYY-MM-DD HH:MM:SS"` with full range validation |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.parse` | integer CLI argument |
| `sys` | `now_ms`, argv — no capability line required |

Run the tests: `dunder test`.
