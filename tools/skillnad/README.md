# skillnad — line diff

**CLI:** `skillnad` · **Library:** `import skillnad` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Skillnad* is Swedish for "difference". `skillnad` compares two text files
line by line with a classic LCS dynamic program and prints a unified-lite
view: `-` lines only in the first file, `+` lines only in the second, in edit
order, followed by a counts summary.

## CLI

```sh
skillnad old.txt new.txt
```

```text
$ printf 'alpha\nbeta\ngamma\n'  > a.txt
$ printf 'alpha\nBETA\ngamma\ndelta\n' > b.txt
$ skillnad a.txt b.txt
-beta
+BETA
+delta
1 removed, 2 added
```

Identical files print nothing and exit 0. Inputs are capped at 2000 lines
each (the DP table is ~4M cells; larger inputs error out with exit 1).

## Library

| Item | Meaning |
| --- | --- |
| `DiffLine { kind, text }` | `kind`: 0 same, 1 only in A (minus), 2 only in B (plus) |
| `diff_lines(&[]string, &[]string) -> Result<[]DiffLine, string>` | LCS diff in edit order; `Err` when either input exceeds 2000 lines |
| `render(&[]DiffLine) -> string` | Only `-`/`+` lines plus a `"<n> removed, <m> added"` summary; empty string when identical |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.text` | trailing-newline handling, line splitting |
| `sys` | file reads — no capability line required |

Run the tests: `dunder test`.
