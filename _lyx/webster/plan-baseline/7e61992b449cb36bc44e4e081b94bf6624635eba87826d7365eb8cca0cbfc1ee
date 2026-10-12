# Card 12 — bare-temp-scan

**Create:**
- `cmd/lyx/baretemp_test.go`
- `pattern/PATTERN-lyx-temp-root.md`

**Edit:**
- `PATTERN.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/testkit/scankit#Walk`

**Intent:**
Add the scan that fails a bare system-temp creation outside the temp package (D6), modelled on `cmd/lyx/hermeticenv_test.go` and `cmd/lyx/tmuxisolation_test.go`.

`baretemp_test.go` walks every Go file of the module, production and test, through `scankit.Walk`, parses each, and reports a call `<os>.MkdirTemp(...)` or `<os>.CreateTemp(...)`, where `<os>` is the name the file binds the `"os"` import to, whose first argument is the string literal `""` (either quote form) or the call `<os>.TempDir()`.
Files under `internal/lyxtemp/` are exempt.
A mention in a comment or a string is not a call, so it never matches; `t.TempDir()` and a dir argument naming any other directory, a variable or a call such as `os.Getenv(...)` included, stay allowed.
A finding names the file, the line and the way forward: create it through `lyxtemp.MkdirTemp` or `lyxtemp.CreateTemp`.
An allowlist in the test file, `[]scankit.Entry` keyed `<file>:<call>` (for example `internal/x/y.go:os.MkdirTemp`), admits one creation call in one file, with a reason naming the obstacle; the allowlist starts empty, since cards 2 to 6 moved every site, and its comment says that an admitted site still removes what it creates on normal exit and that what an entry lets through is that one site's directory left in the bare system temp directory after a kill.
It carries two top-level tests: the guard over the tree, which requires a scan floor and no stale allowlist entry, and a fixture-tree test like `TestTmuxIsolation_FixtureTrees`, with rows for `os.MkdirTemp("", p)`, `os.CreateTemp("", p)`, each with `os.TempDir()` as dir, an aliased `os` import, a mention in a comment, `t.TempDir()`, a named dir, and a bare call inside `internal/lyxtemp/`, each with its expected finding count.

`PATTERN.md` gains, under "Paths and geometry", this entry line:

```markdown
- `PATTERN-lyx-temp-root` — Creating a temp file or directory: under `internal/lyxtemp`'s process run directory; `os.MkdirTemp`/`os.CreateTemp` never take `""` or `os.TempDir()` as dir elsewhere. (test) — [background](pattern/PATTERN-lyx-temp-root.md)
```

`pattern/PATTERN-lyx-temp-root.md` states the per-user root, the locked run directory per process, the sweep and its grace and bound, the beside-the-parent rule for an inherited run directory, that `tmuxkit.Main` and `lyx`'s `main` remove the run directory on normal exit, the scan in `cmd/lyx/baretemp_test.go` and its allowlist, and the bound that a killed process's directory waits for the next start's sweep.
The overview stays under its headroom: the entry adds about 230 bytes to a file at about 15,000 of its 15,564-byte headroom line.
`cmd/lyx/testdata/test-budget.yaml`'s `cmd/lyx` row grows by the two new top-level tests.

**ImpactSummary:** A new repo-wide guard and its PATTERN entry; the tree already passes it, so no production code changes.
