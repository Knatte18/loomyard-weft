---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: contracts/stencils/darn/darn-fragment-make-change.md:15
    summary: The new stencil text and the no-test line tell agents to run a raw dotnet test, which a hub whose refuse_raw lists dotnet test has its shell guard refuse.
    fix: In darn-fragment-make-change.md (lines 15-16), webster-body-implementer.md (lines 117-118) and the way-forward clause of noTestNotice in internal/gatecli/tierrun.go, stop offering a hand run as an unconditional step. Either name a step the guard allows, or state that the hand run applies only while the hub's refuse_raw does not list dotnet test and otherwise tell the agent to report the skipped tests to its parent. Keep the stencil's "never a raw test command" rule consistent with whichever wording is chosen, and update the TestRunTierNoTestNotice expected line to match.
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/gatecli/tierrun.go:353
    summary: The board entry asks for a report of which test classes the gate skipped and why, but the line fires only when a whole invocation reported zero tests, so a whole-run solution invocation where one project is disabled and another runs passes silently, and no skipped class or reason is named.
    fix: In noTestNotice and runInvocations, detect a disabled test project inside a multi-project invocation, for example by comparing the projects the scope selected with the projects whose summary lines the vstest counter read. Name the skipped projects or classes in the line. If the setting that causes the skip cannot be known to the gate, say so in the line, and in internal/gatecli/doc.go state which skips the line covers and which it cannot see.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/tierrun_test.go:231
    summary: TestRunTierNoTestNotice has no row for a failing vstest invocation, which must print no line, nor for an invocation with no targets, whose line names the run directory.
    fix: Add two rows to the table in TestRunTierNoTestNotice. One uses a command that exits non-zero and wants no line. The other leaves targets empty and wants the notice to name the run directory.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/weftwiring.go:62
    summary: The CodeLocation branch that returns a records location unchanged when its paired code directory is absent has no test row.
    fix: Add one row to the refusal table of TestGateTestMap in internal/gatecli/testmap_integration_test.go, or one case to a fabricengine test, that resolves a records sibling whose code directory is removed and wants the location unchanged.
---

### [MEDIUM] A raw dotnet test is offered as a way forward where the shell guard may refuse it

**Location:** `contracts/stencils/darn/darn-fragment-make-change.md:15`, `contracts/stencils/webster/webster-body-implementer.md:117`, `internal/gatecli/tierrun.go:377`
**Issue:** The stencils say "this is the one raw test command you run", and the no-test line's way forward is a `dotnet test <project> -p:<property>=true` run.
The shell guard's refuse-raw row denies any command starting with a prefix in the hub's `gate.yaml` `refuse_raw`, and its notice names `dotnet test` as an example (`internal/shellguard/rows_test.go:67`).
On such a hub the agent that receives the line is refused when it follows it.
`PATTERN-refusal-way-forward` says no refusal, stencil or spec names a step agents' settings deny, and `internal/shuttleengine/claudeengine/doc.go:558` shows the repository naming a step only while `refuse_raw` does not refuse it.
I could not confirm that any hub lists `dotnet test`, so this is MEDIUM.

### [MEDIUM] The no-test line covers a whole-invocation zero, not the skipped classes the entry asks for

**Location:** `internal/gatecli/tierrun.go:353`
**Issue:** The first bullet of the entry asks the gate to report which test classes it skipped and why.
`noTestNotice` reads only the delta of the vstest counter's discovered count across one invocation.
A whole run is one `.sln` invocation, so a disabled project beside a project that ran leaves the delta above zero and prints nothing.
A narrowed project that runs some tests and skips a gated class prints nothing either.
The line never names a skipped class or the setting behind the skip.
The change description admits the line is advisory and zero-test only; the entry's stated bug, a project that disables itself, is caught only when it is the whole invocation.

### [LOW] Two behaviours of the no-test line have no row

**Location:** `internal/gatecli/tierrun_test.go:231`
**Issue:** `noTestNotice` names the run directory when the invocation has no targets, and `runInvocations` prints no line when the invocation exits non-zero.
Every row of `TestRunTierNoTestNotice` sets targets and passes, so a regression in either goes unseen.

### [LOW] The absent-code-directory branch of CodeLocation has no row

**Location:** `internal/fabricengine/weftwiring.go:62`
**Issue:** `CodeLocation` returns a records location unchanged when the paired code directory is missing, which is its safe-failure path.
The two new testmap rows both have a paired code directory.

## Rejected

- Retried tier repeats the notice once per attempt (code-correctness finding 2): unverified, and a repeated advisory line is no defect.
- Notice may land in the log before an unflushed partial last line (code-correctness finding 3): the vstest summary line is newline-terminated, so the case needs an output the counter never sees from dotnet.
- Sibling stencils family not updated (code-affected-set finding 2): the lens itself found no stencil missed.
- `PATTERN-cwd-resolution` not updated (code-affected-set finding 3): `CodeLocation` is in `fabricengine` and uses `lymcwd.NamedWorktree`, so the entry is met.
- No test runs `redundant` from a records worktree (code-tests finding 3): every verb resolves through the one `resolveHub`, which the testmap rows pin, so a second row would repeat that coverage.
- The testmap refusal needs no change in `hubgeom.PrimeWorktreePath`: its only caller is the refusal, so the entry's second bullet is met by reading, not by a diff change.
- `TestGateTest_DetachedRun` and `TestRedundancyStep` fixes: both remove the named race instead of widening a margin, as the code-correctness lens traced.
