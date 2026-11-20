# Dunder dogfooding notes

Issues hit while writing the dunder-suite tools against **Dunder 0.5.0**
(arm64-macos, installed compiler). Collected for the language repo; each entry
says where it happened and what was expected. Ordered by first encounter.

## Compiler messages

1. **`.Err` arm without payload gives `expected '('` at a confusing position**
   (`rita/tests/suite.ddr`). `Result<[]f64, string>` matched with
   `.Err => { ... }` reports `error[E1000]: expected '('` pointing *inside the
   arm body* rather than saying "variant `.Err` has a payload; bind it as
   `.Err(e)`". The `standard.md` example `match table.take(...) { .Err => ... }`
   only works because that `Err` payload is `()`, which makes the docs example
   actively misleading for `Result<T, string>`. A note in docs + a better
   message would help.

2. **`unknown member 'is_ok'` for `parsed.is_ok`**
   (`rita/tests/suite.ddr`). `docs/standard.md` lists `Result<T, E>` "med
   `.is_ok`, `.is_err`, ..." which reads like field access; they are methods
   requiring `()`. The message could suggest "call it: `parsed.is_ok()`", and
   the docs should show parentheses.

3. **`assignment requires a mutable stored value` points at the assignment, not the declaration**
   (`rita/src/rita.ddr`). Clear enough once known, but the fix is at the
   `:=` declaration several lines up. A note like "'hi' declared immutable at
   line 98" would save a round trip.

4. **`push requires 'move value', .clone(), or a fresh owned expression` when the value was only borrowed earlier**
   (`rita/src/main.ddr`: `numbers.push(a)` after `a` had been passed by borrow
   to `==` / `text.starts_with`). Arg-passing borrows seem to conservatively
   block a later move of the same binding in the same block. In Rust (NLL) the
   borrow ends at the call. Either the rule is stricter than necessary or the
   docs' "lånet gäller till blockslut" applies to argument borrows too — worth
   documenting explicitly either way, since the workaround (`a.clone()` on a
   freshly-returned `sys.arg(i)` string) feels wasteful.

## Unclear or missing APIs

5. **`json.Doc.raw_span` returns *unquoted* text for String nodes**
   (`sil/src/sil.ddr`). `raw_span` is exact source for numbers/bools/containers
   but for strings it is the inner content *with escapes still encoded* and no
   quotes. To print a JSON-valid string I had to hand-slice
   `input[starts[node]-1 .. ends[node]+1]`. A `raw_json(node)` (or documenting
   raw_span's per-kind behavior) would remove the trap.

6. **No documented way to read stdin.** `sys` covers files, stderr, args —
   but not fd 0. Tools fall back to `sys.read_text("/dev/stdin")`, which works
   on macOS/Linux but is a POSIX workaround (untested on wasm, and
   `read_text`'s UTF-8 validation is a silent constraint for binary pipes).
   An explicit `sys.read_stdin()`/`read_stdin_bytes()` seems warranted.

7. **No `std.text.lower/upper` or `replace`.** ASCII case folding for
   `finn -i` had to be hand-rolled byte-by-byte, and histogram label cleanup
   needed a custom `format_edge` (see `rita`). Both are classic stdlib
   string helpers.

8. **Owned slicing of a dynamic array is undocumented.** `&values[a..b]` is
   documented as a borrowed view and `text[a..b]` as a new string, but what
   `files[a..b]` (no `&`) means for `[]string` is not stated anywhere —
   copy? view? I used `.clone()` on the view, which works but feels guessed.

## Gotchas

9. **`dunder test` also runs stdlib's own tests** (they appear as
   `<stdlib/bytes.ddr>::...`). Fine — actually nice — but the count includes
   them, which surprised me when reading "22 passed" for 12 local tests.

10. **f64 `.str()` always prints a decimal point (`12.0`, `45.0`).**
    Predictable, but for CLI labels I expected shortest-int-like output; every
    tool ends up writing its own float-to-label trimmer.

11. **Docs/README example drift:** `docs/http.md` + `README.md` describe the
    JSON API via `Writer`/`field_value` accurately, but the lower-level buffer
    helpers (`field_str`, `begin_obj`, `sep_elem`, `finish_text`) that the
    stdlib itself uses are only discoverable by reading `stdlib/json.ddr`.

_New entries are appended while the remaining tools are written._

## Round 2: finn, spar, lod

12. **`spawn` accepts exactly one safe by-value argument**
    (`finn/src/finn.ddr`). `spawn jobs f(a, b, c)` with borrowed string args
    gives `spawn requires one safe by-value worker argument`. The docs'
    multi-argument example (`spawn jobs work(20)`) only works because the
    second parameter is the scope's bound workspace — that's nowhere near
    obvious. Bundling worker args into one `Sendable`/`Clone` struct works
    fine; the restriction (and the struct idiom) deserves a docs section.

13. **Task workers may not perform ambient I/O — error doesn't point at the culprit**
    (`finn/src/finn.ddr`). My worker called `sys.read_text` transitively; the
    error `task worker has effects outside its isolated runtime (ambient I/O
    or system call)` names the worker but not *which call* in the transitive
    chain (`search_group → search_file → sys.read_text`). The rule itself is
    documented in `docs/tasks.md` (transitiv effektanalys), but finding the
    offending call meant re-reading my own code. Ideally the message includes
    the rejected call site.

14. **Dynamic-array slicing `arr[a..b]` is unsupported — misleading messages**
    (`finn/src/finn.ddr`). `merged = merged[0..cap].clone()` first reported
    `assignment requires a mutable stored value` (it wasn't about mutability
    at all — that came from an unrelated immutable binding on the same line
    number after an edit), and after fixing that, `slice requires a string`.
    A dedicated "array slicing is not supported; use a loop or `array.append`"
    message (or supporting it) would help. Only `&arr[a..b]` (borrowed view,
    documented) and `text[a..b]` exist.

15. **Forwarding a `&mut` parameter needs `&mut out` — auto-deref hides it**
    (`finn/src/finn.ddr`). Inside `fn collect_files(..., out: &mut []string)`,
    the recursive call `collect_files(..., out, ...)` fails with
    `type mismatch: &mut []string / []string` — reading `out` auto-derefs it
    to the pointee. Correct spelling is `&mut out`. Also, the mismatch
    message doesn't say which side is expected vs provided (and it appears in
    other messages too — order "expected / got" is guesswork).

16. **The "freshness" rules around `push` are stricter than any doc suggests.**
    Repeatedly needed `.clone()` for values that are intuitively owned:
    loop bindings from `for m in move batch { out.push(m) }` (fresh per
    iteration, yet rejected), a value whose only prior use was `x.len` or a
    `==` comparison (`numbers.push(a)` after flag checks), and borrowed
    function params moved into struct fields (`pattern: pattern`). The
    message lists the three accepted forms, which is good — but the pattern
    "read anything → must clone" is a persistent tax and worth a docs
    section of its own.

17. **Moving a field out of an owned local struct is fine; moving out of a
    struct behind any indirection is not, with the same message as 16.**
    (`path_arg := positional[0]` → `cannot move out of an indexed element`).
    Consistent, but combined with 16 every CLI ends up sprinkled with
    `.clone()` whose only purpose is appeasing the checker on same-scope data.

18. **`std.text` gaps for CLI work:** no `lower/upper` (hand-rolled
    `ascii_lower` in finn and `upper_ascii` in spar), no `replace`, no
    `contains` (only `find(...) >= 0`), and no `pad_left/pad_right` — every
    tool writes its own padding helper for table columns.

19. **`f"{p.port:5}"`-style width formatting worked for integers, but there
    is no string padding in f-strings**, so aligned tables need manual byte
    padding (see `lod.render_table`, `spar.pad_right`). A `:<5`/`:>5` analog
    for strings would remove a whole class of helper code.

20. **`dunder run` executes the built binary with a different working
    directory than the shell** (`finn`: `../dunder/stdlib` "no such file or
    directory" until absolute paths were used). Worth a line in the docs:
    use absolute paths or resolve with `fs.realpath` in scripts.

21. **Positive surprise worth keeping:** `defer` + `fs.remove_all` in tests
    is a clean fixture pattern, `array.append(&mut dst, &src)` merging task
    results is exactly right, and the enum-payload `match` with per-arm
    early `return` made the path parser pleasant. Not everything was
    friction — the error positions in `sil`'s hand-written parser came out
    precise on the first run.

## Round 3: fraga, valv, hylla, salva, bas

22. **`.Ok(mut x)` in a match pattern is a parse error with a confusing message**
    (`valv/src/main.ddr` → `error[E1000]: expected '('`). Wanting a mutable
    binding out of a match is common (every "open store, mutate, save" flow);
    the only working spell is `.Ok(x) => { mut s := x; ... }`, which the
    error message does not suggest. Related: `.Ok(mut store)` reports
    `match pattern requires an enum` in some positions — both messages are
    far from the actual mistake.

23. **`@resource` values cannot be returned through `match` payloads at all.**
    (`bas/src/bas.ddr`). `Result<Child, string>` destructured with
    `.Ok(child) => try_insert(move child)` fails with `cannot move a borrowed
    pattern binding; use .clone()` — and `.clone()` does not exist for
    resources, so the suggested fix is impossible. The workaround was to
    restructure the API so the resource never crosses a match arm
    (`spawn_into(&mut ChildSet, ...)` inserts internally). Either resource
    payloads should be movable out of owned match values, or the message
    needs a resource-aware suggestion. This shaped the design of an entire
    module — worth a docs section on "owning resources through enums".

24. **Wildcard `.Err(_)` arms work but are undocumented.** Discovered by
    necessity in #23; `syntax.md` never mentions `_` in match patterns.
    Please document (or reject) — it's load-bearing for the pattern above.

25. **`std.process.ChildSet::wait_any` returns a poll-ready count, not a slot**
    (`bas/src/bas.ddr`). From the name I expected "the slot of a child that
    exited" — it's actually `poll()`'s ready-fd count (0 on timeout), and you
    still sweep `try_wait` over slots yourself. Works fine, but the name
    invites the wrong mental model; `poll_any` would say what it does.

26. **Moving a field out of an owned local result works and is great**
    (`mut stream := result.stream`, `mut child := spawned.child`) — but note
    the asymmetry with #23: fields of locals move freely while match payloads
    of the same resource do not. Documenting the boundary would have saved
    the whole #23 detour.

27. **`sys.sleep_ms` returns `()`, so `_ := sys.sleep_ms(10)` is a compile
    error** ("local requires a value"). Consistent once known; a hint in the
    message (`unit value; call it without '_ :='`) would help, since `_ :=`
    is the habit for every other syscall-ish call.

28. **Vendoring observation:** dependency `path` entries resolve relative to
    the *depending* manifest, so `vendor/` + `../vendor/http` works cleanly
    per-tool. Git deps can't point at a subdirectory of a repo, so vendoring
    was the only way to make this suite self-contained — a `subdir = ...`
    field for git sources would remove the need.

29. **Positive surprises, round 2:** `fs.temp_dir` + `defer { remove_all }`
    kept ten test suites leak-free; the single-threaded listener/client trick
    (connect → write → accept → handle → read) worked first try for hylla's
    full HTTP round trip and needs no harness; `f"{tenths / 10}.{tenths % 10}"`
    integer formatting covered decimal rendering without float printing; and
    `route()` as a pure function over `(root, upload, method, target)` made
    the HTTP server's security decisions testable without sockets.

## Round 4: fifty small tools built by five parallel agents

The 50 `tools/` projects were written by subagents working from
`AGENT-GUIDE.md`. Their findings (verified against the built tools):

30. **Assigning a string match-payload binding still requires `.clone()`.**
    Even `failure = e` inside a nested `.Err(e)` arm needs `e.clone()` when
    the target outlives the arm — the payload binding is a borrow regardless
    of how innocuous the use is. (Consistent with #23, but surprising in the
    plain-assignment case.)

31. **`i64` arithmetic overflow aborts the process at runtime.** `fibo`'s
    F(93) guard had to avoid *computing* the value even on a final loop
    iteration. Fine for a checked language, but CLIs doing user-supplied
    arithmetic want a checked-multiply/add story or an overflow flag the way
    `i32()` narrowing checks.

32. **Two byte worlds: `[]i32` vs `[]u8`.** `text.bytes()` and
    `sha256.digest`/`sha256.text` live in `[]i32`, while `std.bytes`,
    `std.crypto`, `sys.read_u8/write_u8` and sockets live in `[]u8`. Every
    hashing pipeline needs explicit conversion loops at the boundary
    (hit independently by the `kolla` and `dublett` agents). A unifying
    conversion helper (or `[]i32` overloads) would remove a whole class of
    glue code. *Post-script:* the stdlib source also has
    `sha256.digest_bytes(&[]u8)` — undocumented in `docs/standard.md` —
    which `kolla` now uses directly. The split remains, but the u8-native
    digest deserves a docs mention.

33. **Bare calls to instantiated generic helpers fail with a mangled-name
    error.** `array.reverse(&mut v)` fails where
    `array.reverse<string>(&mut v)` compiles; same for `array.contains` /
    `array.index_of`. The diagnostics reference mangled internal names.
    Either inference should work at call sites, or the message should
    suggest the explicit type argument.

34. **`map.Map` is opaque-ish in practice:** `.len` is a method (`m.len()`),
    and `Map` cannot be constructed without `::new()`. Fine, but distinct
    from plain structs — worth a line in `docs/standard.md`.

35. **`json.Doc::parse_text` rejects literal control characters in strings**
    (spec-correct). `packa`'s tests had to use `\n` escapes in fixtures.
    Worth documenting since shell heredocs often carry raw newlines.

36. **No module-level constants**: a fixed wordlist must be exposed as
    `fn wordlist() -> []string`. Const arrays (or `const` bindings of
    literals) would make table-driven tools cleaner.

37. **`fs.list_dir` requires `capability filesystem`** even though plain
    `sys` reads/writes do not — three separate agents hit this from
    different angles (`granska`, `tom`, and the guide itself). The capability
    boundary between `sys` file I/O and `std.fs` listing deserves a docs
    table: which operation needs which capability.

38. **The single-threaded loopback pattern scales.** The `eko` agent reused
    the hylla listener/client trick for a full TCP echo round trip in a
    `@test fn` with no harness — pattern proven twice now.

39. **Agent-guide lesson:** a written, gotcha-first cheat sheet
    (`AGENT-GUIDE.md`) let five independent agents ship 50 compiling,
    test-passing tools (~693 tests) with zero syntax dead-ends that the
    guide already covered. Every failure they hit was an undocumented
    behavior — which then became a numbered entry here. Dogfooding via
    agents is an efficient way to triage documentation gaps.
