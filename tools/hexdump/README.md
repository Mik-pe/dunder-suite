# hexdump — file hex viewer

**CLI:** `hexdump` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

Renders a file as `OFFSET  hex bytes  |ascii|` rows: 8-hex zero-padded
offset, space-separated hex bytes, and printable ASCII between pipes
with non-printable bytes shown as `.`. Reads raw bytes via
`sys.read_u8`; files larger than 16 MiB are rejected with an error.

## CLI

```sh
hexdump FILE
hexdump --width 8 FILE
```

```text
$ hexdump --width 8 sample.bin
00000000  48 69 01 7f 21 00 00 00  |Hi..!...|
00000008  00 00 00 00 00 00 00 00  |........|
00000010  00 65 6e 64              |.end|
```

`--width` sets bytes per row (1..64, default 16); short rows are padded
so the ASCII column stays aligned.

## Library

```dunder
import hexdump

fn main() -> i32 {
    mut row: []u8 = []
    row.push(b'A')
    print(hexdump.render_row(0, &row, 8))    // 00000000  41 ...  |A|
    return 0
}
```

| Item | Meaning |
| --- | --- |
| `render_row(offset: i64, row: &[]u8, width: i32) -> string` | One padded `OFFSET  hex  |ascii|` row |
| `dump(bytes: &[]u8, width: i32) -> string` | Full dump, one row per `width` bytes (width clamped to 1..64) |

## Dunder capabilities used

None. File reads use `sys.metadata` / `sys.read_u8` (no capability line
required for `sys` reads).

Run the tests: `dunder test` (7 suite tests + stdlib tests).
