---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/boardengine/holdwrite.go:403
    summary: "FixHold's way forward names only the failing subject, so for a multi-subject hold it names a re-run that the same-set rule refuses"
    fix: "In FixHold, build the way forward from the hold's own subjects, taken from `want.Subjects` (or the own hold read inside the write closure) via `holdSubjectNames`, not from the keys of `tries`, at the refusal for no hold standing (line 403) and at the refusal for a hold not standing as written (line 439). Add a row to the FixHold test with a two-subject hold where the second subject fails and the hold is lost, asserting the way forward names both subjects."
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: design
    location: internal/fabricengine/takeoverwrite.go:264
    summary: "rewriteOverOrigin wraps a nil err with %w when the branch has no upstream, so the message prints a bad-verb marker and does not say the upstream is missing"
    fix: "Split the check in `rewriteOverOrigin`: return the existing wrapped error joined with `b.dropEntry(rec)` only for a non-nil err, and for `!hasUpstream` return an error that names the branch as having no upstream, joined with `b.dropEntry(rec)`."
    origin: lens:code-correctness
  - id: F3
    severity: NIT
    class: consistency
    location: internal/adoptcli/cli.go:39
    summary: "The help says a plain adopt waits while another machine holds the hub, but D6 and the code refuse it, naming the writer machine and the way forward"
    fix: "In the `Long` text of `Command`, replace the sentence saying a plain adopt waits on another machine's hold with wording that says a plain adopt refuses while another machine holds the hub, naming that machine and its time."
    origin: handler
---

### [MEDIUM] FixHold's way forward names only the failing subject

**Location:** internal/boardengine/holdwrite.go:403, 439; fed by internal/adoptcli/fixloop.go:290 and `hubFixDeps`.
**Issue:** `FixHold` derives `subjects` from the keys of `tries`, and the fix loop passes the failing subject's count alone.
For a hold over `a b` where `b` fails, a hold that is lost or missing meanwhile yields the way forward `run lym adopt b`.
`planAdoptHold` refuses a re-run over any other set than the hold's, so the named command does not work, against `PATTERN-refusal-way-forward`.

### [LOW] rewriteOverOrigin wraps a nil error

**Location:** internal/fabricengine/takeoverwrite.go:264-266.
**Issue:** `if err != nil || !hasUpstream` returns a message built with `%w` over `err`, which is nil when the branch only lacks an upstream.
The message then prints `%!w(<nil>)` and hides the cause of the failed retry.

### [NIT] The help describes a wait that the code does not do

**Location:** internal/adoptcli/cli.go:39.
**Issue:** The `Long` text says a plain adopt "waits on" another machine's hold.
`WriteAdopt` refuses at once, and its way forward tells the caller to wait.

## Rejected

- code-tests F1 (no test asserts the take-hold row's steer and notice omit the flag): the steer and notice are static prose constants, and the rubric counts a new assertion on static prose as a test-economy finding, so adding one would be a finding itself.
  The code and the shellguard init check hold the property, and the affected-set lens confirmed the wording.
- code-tests F2 (no negative coverage for the retired items): the lens itself marks it not a finding; deletions are checked by assert-no-callers, and the affected-set lens found no caller left.
