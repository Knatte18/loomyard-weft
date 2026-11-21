---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/websterengine/verifygate_test.go:248
    summary: The new note reader repeats a filename instead of using its single named declaration.
    fix: Declare the webster verify triage note stem once as a named constant in triage.go; use it in the writer and derive the filename from that constant in triage_test.go and verifygate_test.go.
    origin: lens:code-affected-set
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/websterengine/triage.go:101
    summary: A backtick fence in captured output closes the diagnostic block and turns subsequent output into Markdown.
    fix: In writeFailureOutput choose a fence longer than every backtick run in the tail and use it for both delimiters; extend the existing note-content test with embedded fences and following diagnostic text.
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/websterengine/verifygate_test.go:218
    summary: The first-run provenance test supplies identical output for both verify runs.
    fix: Give the existing TestVerifyGate_FlakyFailurePassesOnRerunWithNote table fixture distinct first-run and passing-rerun log contents; retain the first-run diagnostic checks and assert the rerun-only text is absent from the note.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: consistency
    location: contracts/specs/webster-spec.md:72
    summary: Documentation promises command classification for package-only failures that the code still reports as flaky packages.
    fix: Qualify the new classification statements in webster-spec.md, websterengine/doc.go and the triageWarnings comment to say that neither a failing test nor a failing package was named; retain the existing package classification.
    origin: lens:code-affected-set
---

### [BLOCKING] Declare the note name once

**Location:** internal/websterengine/verifygate_test.go:248
**Issue:** The added reader spells `webster-verify-triage.md` independently of the writer's stem in `triage.go:63`.
The existing reader in `triage_test.go:61` repeats the filename too.
`pattern/PATTERN-code-conventions.md:11` requires one named declaration for a name or path that can go stale.
The round's constraints make every violation of that rule BLOCKING.

### [MEDIUM] Preserve fences inside diagnostic output

**Location:** internal/websterengine/triage.go:101
**Issue:** The writer surrounds arbitrary captured output with a fixed three-backtick fence.
For an opaque tail containing `before\n```\n# diagnostic\n```\nafter`, the copied fence closes the block.
The heading becomes Markdown, and later fences can absorb the note's following text.
`ParseVerifyRun` can supply this opaque tail unchanged; ordinary multiline and empty tails are handled correctly.

### [MEDIUM] Distinguish first-run output from rerun output in the regression fixture

**Location:** internal/websterengine/verifygate_test.go:218
**Issue:** The fixture supplies only `tt.firstLog`, and `gateFake.seams` repeats its last log for subsequent calls.
The first and second log paths therefore return identical bytes.
The assertions would accept a regression that kept the first log path but read the tail from the passing rerun.
This is a coverage gap in the new provenance guarantee, not a request for another test function.

### [LOW] Describe the opaque classification precisely

**Location:** contracts/specs/webster-spec.md:72
**Issue:** The new sentence says every run naming no failing test is reported as a command failure.
`ParseVerifyRun` also creates package identities without a named failing test.
`commandFailure` recognizes only opaque failures, and the changed gate test explicitly retains flaky-package reporting.
The parallel statements in `doc.go:993` and `triage.go:41` carry the same mismatch.

## Rejected

- The scope lens's cwd-notice finding assumes the outer harness's decoration appears in subprocess output.
  `verifytree.runAndRecord` captures the subprocess into its log through `verifyrun.Run`.
  `NewVerifyGate` reads that file directly, without an agent tool-result envelope.
  An injected notice would survive a tail, but no production path injecting the harness notice was established.
  An ordinary notice also cannot become a test or package identity under the parser's line grammar.
- Deduplication loss was considered and rejected for the current caller.
  The parser emits at most one opaque identity and deduplicates each first run before `addFlaky`.
  Webster appends verify last; its successful evaluation ends the gate sequence and is memoized by shuttle.
  No second successful flaky evaluation accumulating a different log in the same notes was established.

## Focus checks

All focus items were inspected.
Opaque failures remain apart in warnings, notes and summaries; package identities retain their existing category.
Trailing blank output lines do not break indentation, and empty output receives an explicit sentence.
The spec revert/reapply leaves the net spec update present, consistent with the parent's red-first waiver.
The whole-tree symbol search found no remaining `.Flaky(` caller.
The removed identity assertion is replaced by an exact warning assertion through `Apply`.
The changed expected values are independent literals, and the added cases introduce no real-time waits.
The reedcli explanation traces to the tmuxkit attribution change already present in the review base.

## Validation

Attempted `timeout 3600s lym gate test --tier 0 ./internal/websterengine` in the foreground.
The gate could not acquire its slot because the shared `_bolt/.lym/gate` directory was read-only.
Its process registration also reported a read-only lock directory.
No test ran in this attempt; this environment limitation is not a code finding.
