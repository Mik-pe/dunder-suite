# validera — JSON validator

**CLI:** `validera` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Validera* is Swedish for "validate". Checks each file as JSON and prints one
line per file: `<path>: ok` or `<path>: error: <message> at byte <offset>`.
Exits 1 if any file is invalid (or unreadable), 0 when everything passes.

## CLI

```sh
validera good.json broken.json
# good.json: ok
# broken.json: error: trailing characters after JSON value at byte 8
```

Every file is checked even after a failure; the exit code reflects the whole
run.

## Library

| Item | Meaning |
| --- | --- |
| `validate(text) -> Result<(), string>` | `Ok` for valid JSON; `Err` carries `"<error> at byte <offset>"` ready to print |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.json` | `Doc::parse_text` (`ok`/`error`/`offset`) |

No capabilities required — pure computation, files read via `sys`.

Run the tests: `dunder test`.
