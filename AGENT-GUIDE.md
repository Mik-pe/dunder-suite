# Dunder agent guide — read this BEFORE writing any code

This guide is for agents (and humans) adding tools to dunder-suite. It
distills everything learned implementing the first ten tools against
**Dunder 0.5.0**. The compiler is at `~/.dunder/bin/dunder` (already
installed, reports `dunder 0.5.0`). Work from the dunder-suite root.

See also `DUNDER-ISSUES.md` for the raw findings behind these rules.

## Project layout (every tool, no exceptions)

```
tools/<name>/
├── README.md          # title, status line, purpose, CLI usage, library API table
├── dunder.toml        # name/root/test_root
├── src/<name>.ddr     # library: pure logic where possible
├── src/main.ddr       # CLI: `import <name>` + arg parsing + print/eprint
└── tests/suite.ddr    # `import ..src.<name> as <name>` + @test fns
```

`dunder.toml`:

```toml
name = "<name>"
root = "src/main.ddr"
test_root = "tests/suite.ddr"
```

**Verify every tool before reporting done** (from the tool's directory):

```sh
cd tools/<name>
~/.dunder/bin/dunder check src/main.ddr   # must say "checked"
~/.dunder/bin/dunder run src/main.ddr -- <example args>   # smoke test
~/.dunder/bin/dunder test                 # must end "N passed, 0 failed"
```

Iterate until all three pass. The test runner also runs stdlib tests — that is
normal. If a tool legitimately needs a missing library, simplify the tool
instead of leaving it broken.

## Language crash course (verified syntax)

```dunder
// declarations
count := 42                    // inferred, immutable
mut total := 0                 // MUST be mut to assign/+= later
total += 1
x: i64 = 7                     // explicit type

// functions: explicit param/return types; last expr = return value
fn add(a: i32, b: i32) -> i32 { a + b }
fn main() -> i32 { return 0 }  // main() may return i32 (exit code)

// control
if a > b { ... } else if a == b { ... } else { ... }
while i < n { i += 1 }
for i in 0..n { ... }          // half-open range
for value in values { ... }    // borrows array/strings; value is NOT movable
for r in text.runes() { ... }  // rune iteration
match result {
    .Ok(v) => { ... },         // payload MUST be bound (except Result<(), E>)
    .Err(e) => { ... },        // payload MUST be bound; `.Err(_)` also works
}

// enums with payload
enum Segment { Key(string), Index(i32) }
match seg { .Key(name) => {...}, .Index(n) => {...} }   // list ALL variants

// structs: literal needs EVERY field
pub struct Options { width: i32, labels: bool }
opts := Options { width: 40, labels: false }

// strings: immutable UTF-8; + concatenates; s[a..b] is a NEW string
f"{name}: {count + 1}"         // interpolation; ints as :d/:x/:X with width f"{v:04X}"
value.str()                    // f64 prints "12.0"; ints print plain

// arrays
mut values: []f64 = []
values.push(1.5)               // push needs a FRESH value (see rules)
values.len / values.pop() / values.swap(i,j) / values.clear()
fixed: [i32; 4] = [0; 4]

// bytes
mut buf: []u8 = []
buf.push(b'A')                 // byte literal b'x'
bytes.append_utf8(&mut buf, "text")
bytes.decode(&buf).value       // -> string

// early returns inside loops/fns are fine; assert(cond) for tests
```

## The ten rules that will bite you (all verified the hard way)

1. **`mut` at declaration, not at use.** `x := 0; x = 1` fails with
   "assignment requires a mutable stored value". Declare `mut x := 0`.

2. **`push` needs a fresh value.** If a binding was read *any* way before
   (compared, `.len`, printed, borrowed to a call), `arr.push(v)` fails with
   "push requires 'move value', .clone(), or a fresh owned expression".
   Fix: `arr.push(v.clone())` (strings share text; clone is cheap).

3. **Never move out of: array elements (`arr[i]`), borrowed params, or match
   payload bindings into another call's `move` parameter.** Always `.clone()`.
   You CAN move: a field of an owned local (`s := result.value`), and a local
   that has never been read.

4. **Match arms**: payload enums must bind (`.Ok(x)`), no exceptions except
   `Result<(), E>` where bare `.Ok` works. `mut` in patterns is illegal; to
   mutate a matched value: `.Ok(s) => { mut s2 := s; ... }`. List every
   variant of your own enums; only `_` payload wildcards inside a variant
   (`.Err(_)`) are known to work.

5. **Resources (`TcpStream`, `Child`, sockets) cannot leave a match arm at
   all.** If `Result<Child, string>` needs its payload moved on, restructure:
   keep the resource in one function (`spawn_and_insert(&mut set, ...)`).

6. **Dynamic array slicing `arr[a..b]` does not exist** ("slice requires a
   string"). Copy with a loop. String slicing `s[a..b]` is fine.

7. **Borrowed params cannot fill struct literal fields**: write
   `pattern: pattern.clone()`, `args: args.clone()`.

8. **`for x in text.split(...)`** — bind first: `lines := text.split(s, "\n")`
   then `for line in lines`.

9. **Unit-returning calls cannot be discarded**: `sys.sleep_ms(10)` fine,
   `_ := sys.sleep_ms(10)` is an error. Every other syscall-ish result should
   be consumed or `_ :=`.

10. **`dunder run` executes with a different working directory** — smoke tests
    with files should use absolute paths from your shell (`$PWD/...`).

## stdlib cheat sheet (signatures verified in this repo)

```dunder
import sys    // NO capability needed for these
sys.read_text(path) -> {ok, value, error}       // validates UTF-8
sys.write_text(path, text) / sys.append_text(path, text) -> {ok, error}
sys.read_u8(path) / sys.write_u8(path, &[]u8)   // raw bytes
sys.arg_count() -> i32;  sys.arg(i) -> string   // argv; arg(0) = program
sys.eprint(text)                                // stderr + newline
sys.now_ms() -> i64;  sys.sleep_ms(ms)          // unit!
sys.metadata(path) -> {ok, size, ...};  sys.file_kind(info) -> .File/.Directory/.Other
sys.mkdir(path) / sys.remove_file(path) / sys.rename(from, to)
sys.error_message(code) -> string;  sys.decode_u8(&buf) -> {ok, value}
sys.read_text("/dev/stdin")                     // reading stdin works this way

import std.fs   // requires: capability filesystem
fs.list_dir(path) -> {ok, entries: []string, error}   // names only, sorted
fs.temp_dir("prefix-") -> {ok, value}                 // create + cleanup with remove_all
fs.remove_all(path) / fs.mkdir_all(path) / fs.cwd() / fs.realpath(path) / fs.is_symlink(path)

import std.text
text.starts_with(s, prefix) / text.ends_with(s, suffix) -> bool
text.find(haystack, needle) -> i32          // byte index or -1
text.split(s, sep) -> []string              // preserves empty parts
text.trim(s) -> string                      // ASCII whitespace

import std.parse
parse.integer(s) -> {ok, value: i64, error}             // error 1 spelling, 2 overflow
parse.value<i32>(s) / parse.value<i64>(s) / parse.value<f64>(s) -> Result<T, string>

import std.array
array.sort<T: Ord>(&mut values)             // strings: UTF-8 byte order
array.sort_by<T>(&mut values, fn(&T, &T) -> Ordering)   // pass a named fn
array.reverse<T>(&mut values)               // MAY need explicit instantiation: array.reverse<string>(&mut v)
array.contains<T>(&values, &x)              // likewise: array.contains<string>(&v, &x)
array.index_of<T>(&values, &x) -> i32
array.join<T: Display>(&values, sep) -> string

import std.math
math.min(a, b) / math.max(a, b) / math.clamp(v, lo, hi)     // f64
math.total_cmp_f64                                           // fn(&f64,&f64)->Ordering

import std.bytes
bytes.append_utf8(&mut buf, s); bytes.decode(&buf) -> {ok, value}
bytes.hex_u8(&buf) -> string                // encode hex; decode lives in std.crypto
bytes.find_byte(&buf, start, end, b) -> i32; bytes.append_decimal(&mut buf, i64)

import std.json
mut w := json.Writer::new()                 // w.object() w.key("k") w.string(s)
w.integer(i64) w.decimal(f64) w.boolean(b) w.null() w.end() w.finish() -> {ok, value}
doc := json.Doc::parse_text(s)              // doc.ok, doc.error, doc.offset, doc.root
doc.kind(node) -> .Object/.Array/.String/.Number/.Bool/.Null
doc.field(obj, key) -> node; doc.at(arr, i) -> node; doc.count(node) -> i32
doc.as_str(node) / doc.as_i64 / as_f64 / as_bool -> {ok, value}
doc.key_at(obj, i) / doc.val_at(obj, i)
doc.raw_span(node)  // CAUTION: strings return UNQUOTED content; use json.quote(s) instead
json.quote(s) -> string

import std.toml
toml.parse(text) -> {ok, error, json: string}   // TOML -> JSON text

import std.sha256
sha256.digest(&[]i32) -> []i32   // 32 bytes; NOTE: []i32 world, NOT []u8 — convert loops
sha256.text(s) -> []i32          // render with bytes.hex_u8? NO — hex_u8 is []u8; convert i32->u8 first

import std.crypto  // requires: Homebrew openssl@3 + argon2 present here
crypto.random(count: i32) -> {ok, value: []u8, error}
crypto.hmac_sha256(&key, &data) -> []u8
crypto.hkdf_sha256(&key, &salt, &info, len) -> {ok, value, error}
crypto.aes256_gcm_seal(&key32, &nonce12, &aad, &pt) -> {ok, ciphertext, tag, error}
crypto.aes256_gcm_open(&key32, &nonce12, &aad, &ct, &tag) -> {ok, value, error}
crypto.argon2id_hash(password) -> {ok, value: PHC-string}
crypto.argon2id_verify(password, phc) -> bool
crypto.decode_hex(s) -> {ok, value: []u8}; crypto.equal(&a, &b) -> bool

import std.map as map   // Map<K, V>::new()  K: Hash
m.put(move key, move value) / m.at(key) -> borrowed / m.contains(key)
m.remove(key) -> bool / m.keys() -> []string (clones) / m.len()   // len() is a METHOD

import std.net   // requires: capability network
net.SocketAddr { ip: .V4(127,0,0,1), port: u16, scope_id: 0 }
net.TcpListener::listen_addr(addr, backlog) -> {ok, listener}
listener.accept() -> {ok, would_block, stream}; net.local_port(&listener) -> i32
net.TcpStream::connect_addr(addr) -> {ok, stream}
net.write_all(&stream, &data) -> {ok}; stream-or-net.read_append(&mut buf, max) -> {ok, eof, count}
stream.close()

import std.process  // requires: capability process
process.Command::new(program, args); cmd.inherit_stdio(); cmd.exec() -> Output
output.succeeded() / output.code / output.stdout_text()
process.env("PATH") -> {ok, value}; process.host_os() / host_arch() / host_cpus()
```

**Task workers may NOT do any I/O** (transitive effect analysis rejects
sys/fs/net/process calls). For these small tools, write single-threaded code.

**Two byte worlds exist:** `text.bytes()` and `sha256.*` use `[]i32`
(values 0–255), while `bytes.*`, `crypto.*`, `sys.read_u8/write_u8` and
sockets use `[]u8`. Convert with explicit loops at boundaries.
**No module-level constants**: expose fixed tables as `fn wordlist() -> []string`.
**`fs.list_dir` needs `capability filesystem`** even though plain `sys` file
reads do not.

## CLI conventions (match the existing ten tools)

- `fn usage() -> i32` prints to stderr, returns 2
- unknown flag → `sys.eprint("name: unknown flag " + a)`, exit 2
- runtime errors → stderr with `name:` prefix, exit 1
- positional values collected with `arr.push(a.clone())` after flag checks
- output via `print(...)` (stdout, newline appended)

## Test conventions

- 3+ `@test fn descriptive_name()` per tool testing the LIBRARY functions
  (pure logic), not the CLI plumbing
- fixtures via `fs.temp_dir("name-suite-")` + `defer { _ := fs.remove_all(dir) }`
- temp-dir paths must be `.clone()`d when stored into structs/arrays
- Result handling pattern that compiles:

```dunder
match thing() {
    .Ok(v) => { assert(v == expected) },
    .Err(e) => { assert(e.len >= 0); assert(false) },
}
```

## README format

```markdown
# <name> — <short description>

**CLI:** `<name>` · **Status:** v0.1.0 — implemented, tests passing · **Target:** Dunder 0.5.0

<purpose paragraph>

## CLI
<usage examples with real output>

## Library
<table of public fns>

## Dunder capabilities used
<modules + capability lines>

Run the tests: `dunder test`.
```
