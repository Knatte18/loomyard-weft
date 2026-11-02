---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/output/summary.go:158
    summary: "A plain list under a note key (warnings, way_forward, a _error key) longer than five items collapses to a count and its text is never printed"
    fix: "In summaryRenderer.list, when isNoteKey(key) holds for the list's key, skip the count collapse and write each non-empty string through r.noteLines(depth, key, items) instead; add a row to TestSummary in internal/output/summary_test.go with a top-level warnings list of six strings that expects one line per warning."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/clihelp/readable_test.go:53
    summary: "No row covers the flush before the cobra error envelope when a verb holds an unterminated line and then fails, nor a command's own --envelope flag shadowing the global one"
    fix: "Add two rows to TestUseReadableOutput. One verb writes the unterminated line {\"ok\":true} and returns an error, and the row asserts the summary line comes before the cobra error envelope. A second operator verb declares a local envelope bool flag, and the row asserts that setting it prints the envelope."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/output/summary_test.go:33
    summary: "The five-item collapse boundary and the note paths outside a collapsed object (a _error key, a scalar _warning, a way_forward at depth zero or one) have no row"
    fix: "Add a five-item plain list to LongPlainList_CollapsesToCount that expects one joined line and no footer, and add a TestSummary row with a failed envelope whose nested object carries a way_forward and an x_warning that expects both printed on their own lines by path."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-cli-cobra.md:19
    summary: The CLI pattern background documents the global json flag and its shadowing but not the new global envelope flag or the operator summary
    fix: "Add a bullet after line 19 of pattern/PATTERN-cli-cobra.md saying the global envelope flag, declared by clihelp.InstallReadableOutput, keeps the full JSON envelope where an operator verb on a terminal prints its readable summary, and that a command's own local envelope flag, such as lym creel read, shadows it."
  - id: F5
    severity: LOW
    class: consistency
    origin: handler
    location: internal/clihelp/readable.go:25
    summary: WriterIsTerminal is a second terminal-writer check beside the existing writerIsTerminal in shedverbs, with a different mechanism
    fix: "Make one terminal-writer check the single one. Either export clihelp.WriterIsTerminal and have internal/shedverbs/human.go build its writerIsTerminal variable from it, keeping the test seam, or document in the WriterIsTerminal comment why the shedverbs check differs."
---

### [MEDIUM] Long plain list under a note key loses its text

**Location:** `internal/output/summary.go:158` (`summaryRenderer.list`, reached from `field`).
**Issue:** `list` returns after printing `key: N items` for a plain list longer than `summaryListLimit`, and `notes` never runs on it.
A stop whose `warnings`, `way_forward` or `*_error` value is a list of more than five strings prints only a count.
That contradicts the entry's "the way forward for each stop, one line each" and the summary's claim that every error, warning and `way_forward` still prints on its own line at any depth.
The deep-object test covers `warnings` only inside a collapsed object, so no row would fail.
I confirmed the branch by reading `list`: the collapse sits before any note handling.

### [LOW] Flush on a cobra error and a local `--envelope` flag are untested

**Location:** `internal/clihelp/readable_test.go:53`.
**Issue:** `exec.go` now runs the finish steps before the cobra error envelope, and `UseReadableOutput` honours a command's own `--envelope` flag (`lym creel read`, `lym loom arbiter`).
`TestUseReadableOutput` covers only the success path and the global flag, so deleting either behaviour passes every row.

### [LOW] Collapse boundary and note paths lack rows

**Location:** `internal/output/summary_test.go:33`.
**Issue:** The threshold test uses six and two items, so `>` against `>=` is unpinned, and note keys at depth zero or one outside a collapsed object are never rendered in a test.

### [LOW] PATTERN-cli-cobra background omits the new global flag

**Location:** `pattern/PATTERN-cli-cobra.md:19`.
**Issue:** The file describes the global `--json` flag and shadowing by a local flag.
The diff adds a second global flag with the same shadowing behaviour, and the background file does not mention it.

### [LOW] Second terminal-writer check

**Location:** `internal/clihelp/readable.go:25`.
**Issue:** `internal/shedverbs/human.go:19` already declares `writerIsTerminal` for `lym shed status`, using the file's Stat mode.
The diff adds `clihelp.WriterIsTerminal` on `term.IsTerminal`, so two checks now decide "is a terminal" for output, and the summary says `lym shed status` uses the same rule.

## Rejected

- Object lists left uncollapsed, and the five-item plain-list threshold, against the entry's collapse bullet: not a defect. The entry's example is a divergence's commit hashes, which are plain values and collapse to a count. Every run's record prints as one line per item with its notes, and the footer points at `--envelope` for the full list.
- Output paths that bypass the swapped writer: not a defect. The only direct `os.Stdout` uses are `reedcli` attach, which hands the terminal to tmux, and two terminal-size lookups; every envelope writer goes through `cmd.OutOrStdout()`.
- `Write` and `Flush` byte counts and ordering: no byte is lost, duplicated or reordered in the cases run, including a split envelope, a non-envelope `{` line and a line that never ends; `Flush` runs in `RunRootCtx` on both the success and the error return.
- `cmd.SetOut` runs although `onFinish` is a no-op without an exit state (`lens:code-correctness` F2): `RunRootCtx` always seeds the exit state, so the case is unreachable in production.
- `{"ok":true,"error":"…"}` drops the error text (`lens:code-correctness` F3): no producer emits it.
- Test row for a failing underlying writer (`lens:code-tests` F3): the writer passes errors straight through, and a row would pin trivial propagation.
- A test that builds the real root in `cmd/lym` (`lens:code-tests` F6): `clitree_test.go` already walks the real `newRoot()`, and a test of the two wiring lines adds little.
- `metaFlags` lists the root's `--envelope` in `lym --json` (`lens:code-affected-set` F2, unconfirmed): `renderCmdJSON` lists only local flags, so only the root shows it, as it shows `--verbose`.
- Cosmetic edges (`failed: ` with an empty error, `<nil>` for a null list item, an object-valued note printed as a field count): no known producer, no operator impact.
