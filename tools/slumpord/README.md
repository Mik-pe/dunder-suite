# slumpord — passphrase generator

**CLI:** `slumpord` · **Library:** `import slumpord` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

*Slumpord* is Swedish for "random word". It builds passphrases from a fixed
40-word Swedish-ish ASCII list, picking each word from `crypto.random`, or
prints a 32-character hex token. Consecutive duplicate words are re-drawn, so
`apa-apa` never appears.

## CLI

```sh
slumpord                     # 4 words joined with '-'
slumpord -w 6                # 6 words
slumpord --sep .             # custom separator
slumpord --hex               # 32 hex chars instead of words
```

```text
$ slumpord -w 3
ost-riddare-damm
$ slumpord --hex
470129396bc3cbe6b790ddc69872894c
```

Entropy: log2(40) ≈ 5.3 bits per word (4 words ≈ 21 bits — use more words or
`--hex`, 128 bits, for real secrets).

## Library

| Item | Meaning |
| --- | --- |
| `wordlist() -> []string` | The 40 built-in words (a function — Dunder 0.5.0 has no module globals) |
| `pick_words(count: i32, sep: string) -> Result<string, string>` | `count` in 1..64, joined with `sep`; consecutive repeats re-drawn |
| `hex_token() -> Result<string, string>` | 16 random bytes as 32 lowercase hex chars |

## Dunder capabilities used

| Area | What |
| --- | --- |
| `std.crypto` | `crypto.random(4)` per word, `random(16)` for hex tokens |
| `std.bytes` | hex rendering |

No capability line required. Run the tests: `dunder test`.
