# slug — slugifier

**CLI:** `slug` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

Turns arbitrary text into URL-style slugs: ASCII letters are lowercased, runs
of `a-z0-9` are kept, every other rune becomes a single `-`, repeats collapse
and leading/trailing `-` are trimmed. Non-ASCII letters are dropped (they act
as separators).

## CLI

```sh
slug TEXT...     # one slug per argument, or stdin lines when no TEXT
```

```text
$ slug "Some Title Here!"
some-title-here

$ printf 'Ångström Units\nFoo_Bar baz\n' | slug
ngstr-m-units
foo-bar-baz
```

Exit: 0 ok, 1 read error, 2 usage.

## Library

| Item | Meaning |
| --- | --- |
| `slugify(s) -> string` | `"Some Title Here!"` → `"some-title-here"`; unicode-aware dropping |
| `slugify_lines(content) -> []string` | Slugify each line of a document |
| `to_lines(content) -> []string` | Split on `\n`, dropping one trailing empty part |

## Dunder capabilities used

None — `sys`, `std.text`.

Run the tests: `dunder test`.
