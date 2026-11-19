#citera — JSON string quoting

**CLI:** `citera` · **Library:** `import citera` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Citera* is Swedish for "quote". `citera` prints each input line as a valid
JSON string via `json.quote`, and reverses the transformation with
`--unescape` by running the real JSON parser over the line wrapped in
quotes — so escapes like `\n`, `\t`, `\"`, `\\` and `\uXXXX` are handled by
the stdlib, not reimplemented.

## CLI

```sh
citera lines.txt              # quote each line of a file
echo 'say "hi"' | citera      # stdin
citera "first" "second"       # each argument is a line
citera --unescape 'a\nb'      # reverse: prints a, then b on the next line
```

```text
$ printf 'hello\nsay "hi"\n' | citera
"hello"
"say \"hi\""
$ citera --unescape 'm\u00f6n'
mön
```

An argument naming an existing file is read as a file; anything else is a
literal line. With `--unescape`, invalid JSON strings are reported to stderr
with their line number and the tool exits 1.

## Library

| Item | Meaning |
| --- | --- |
| `quote_line(s: string) -> string` | `json.quote` — one valid JSON string literal |
| `unquote_line(s: string) -> Result<string, string>` | Parse `"<s>"` with `json.Doc::parse_text`; `Err` on any invalid JSON |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.json` | `quote`, `Doc::parse_text`, `as_str` |
| `sys` | stdin/file reads — no capability line required |

Run the tests: `dunder test`.
