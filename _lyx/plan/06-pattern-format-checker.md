# Card 6 — pattern-format-checker

**Edit:**
- `internal/pattern/doc.go`

**Create:**
- `internal/pattern/check.go`
- `internal/pattern/check_test.go`
- `internal/pattern/overview_test.go`

**Uses:**
- `PATTERN.md`

**Intent:** Add the PATTERN format checker and run it against loomyard's own overview under `go test` (D2, D8).

`check.go` holds a pure checker over a PATTERN root given as an `fs.FS` (the root holding `PATTERN.md` and `pattern/`).
It reads nothing outside the `fs.FS`, writes nothing, and returns a slice of findings.
Each finding carries a stable kind identifier, the entry name or file it concerns, the overview line number where one applies, and a message.
It reports:
- an entry name not matching `PATTERN-[a-z0-9-]+`, and a duplicated name;
- an entry bullet that continues onto a following line;
- an entry line longer than 320 characters;
- an overview larger than 16 KiB;
- an overview that is empty or whitespace-only;
- an overview with no entry bullet;
- a background link that does not resolve in the `fs.FS`, or does not point at `pattern/<the entry's own name>.md`;
- a file under `pattern/` with no entry naming it.

An absent `PATTERN.md` is its own finding.
The limits and the entry grammar are named constants, each declared once.
Nothing in the prompt-render path calls the checker; `Directive` stays content-agnostic beyond the whitespace rule.
The package stays a leaf: stdlib only, beyond its existing imports.

`check_test.go` is table-driven over `fstest.MapFS` fixtures, one case per finding kind:
- bad name and duplicate name;
- two-line entry and over-long line;
- oversize, empty and entry-less overview;
- unresolved link and link to another entry's file;
- orphan background file and absent overview;
- a clean overview with and without background links, asserting zero findings.

Each case asserts the exact finding kinds and subjects, not only a count.

`overview_test.go` locates the module root by walking up from the test's working directory to the directory holding `go.mod`, then runs the checker on `os.DirFS(root)`.
It never skips.
A missing `PATTERN.md` or any finding fails it, printing every finding.

`doc.go` gains a section documenting the PATTERN format, which is how another repository starts one (D8: no init verb).
It covers the entry-line shape, `(test)`, background links and their naming, the caps, and the rule that PATTERN text states rules for the code only.

**ImpactSummary:** Adds a new pure function and its tests in internal/pattern; no existing caller changes.
