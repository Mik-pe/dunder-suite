# lagra — tiny key-value store on a JSON file

**CLI:** `lagra` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Lagra* is Swedish for "store". A miniature `get`/`set`/`list`/`rm` key-value
store persisted as one JSON object in a single file (default `./lager.json`).
A missing store file starts empty; the first `set` creates the file. Keys
must be non-empty; `set` overwrites; the file always holds keys in sorted
order.

## CLI

```sh
lagra set api-key sekrit        # writes ./lager.json
lagra set host example.com
lagra get api-key               # sekrit
lagra list                      # api-key, host (sorted)
lagra rm host
lagra get ghost --file other.json && echo found
```

Keys and values are stored as JSON strings. `get`/`rm` of a missing key and
any unreadable/broken store file exit 1 with a message on stderr.

## Library

| Item | Meaning |
| --- | --- |
| `Store` | Struct wrapping `map.Map<string, string>` |
| `load(path) -> Result<Store, string>` | Missing file → empty store; invalid JSON, non-object root or non-string values → `Err` |
| `save(&Store, path) -> Result<(), string>` | Writes the store as one JSON object with sorted keys |
| `Store::get(key)` / `set(key, value)` / `remove(key)` | Read, write (rejects empty keys, overwrites), delete |
| `Store::keys_sorted()` / `len()` | Sorted key list, entry count |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.json` / `std.map` / `std.array` | Store parsing (via `sys.read_text`), map, key sorting |
| `std.fs` (tests only) | `fs.temp_dir` fixtures; the library itself needs no capability |

Run the tests: `dunder test`.
