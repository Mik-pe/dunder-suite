# rot13 — Caesar cipher filter

**CLI:** `rot13` · **Library:** `import rot13` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

`rot13` rotates ASCII letters in text by N positions (13 by default, its own
inverse). Digits, punctuation, whitespace and all non-ASCII UTF-8 text pass
through untouched, so it is safe on any text file.

## CLI

```sh
rot13 letter.txt            # classic 13 shift
echo hello | rot13          # stdin
rot13 -n 3 message.txt      # any shift: a->d
rot13 -n -3 message.txt     # negative shifts work (normalized mod 26)
rot13 -n 26 message.txt     # identity
```

```text
$ echo "Hello, World!" | rot13
Uryyb, Jbeyq!
$ printf 'mön\n' | rot13
zöa
```

## Library

| Item | Meaning |
| --- | --- |
| `rotate(text: string, n: i32) -> string` | Rotate a-z/A-Z by `n` (any i32, normalized mod 26); everything else passes through byte-for-byte |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.bytes` | byte-wise rewrite + `bytes.decode` back to string |
| `sys` | stdin/file reading, argv — no capability line required |

Run the tests: `dunder test`.
