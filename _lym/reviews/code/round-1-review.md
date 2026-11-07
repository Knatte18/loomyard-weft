---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/hubforge/template_test.go:161
    summary: The refusing TB's skip refusal (Skip, Skipf, SkipNow) has no test row, though the code, doc.go, the PATTERN background and the summary all state it
    fix: In internal/hubforge/template_test.go add three rows to failingBuilds, built with newFailingBuild, named Skip, Skipf and SkipNow. Each build calls the matching tb method (tb.Skip("x"), tb.Skipf("x"), tb.SkipNow()) and its wantFirst is that method's name. The existing table test then asserts the first caller fails naming the key and the method, a later caller fails naming the key, and the build ran once.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/hubforge/template_test.go:161
    summary: A build that records a non-fatal failure and continues (Errorf, Error, Fail) is never exercised, only the Fatalf path is
    fix: In internal/hubforge/template_test.go add one row to failingBuilds, named Errorf, whose build calls tb.Errorf("the build erred") and returns, with wantFirst "the build erred". The table test then pins that a continuing build still marks the template failed and fails every caller naming the key.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/hubforge/template.go:73
    summary: The Registration.Build comment and the package doc omit the rule that a build reaches no code whose result a once-only registration changes, which the PATTERN background and the decision record both state
    fix: In internal/hubforge/template.go extend the Build field comment after the list of process-wide calls with one sentence, that a build also reaches no code whose result such a registration changes, such as a Bolt replay. In internal/hubforge/doc.go add the same clause to the paragraph that begins "A build's process-wide calls are outside its TB's reach", after "So a build makes none of them."
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: internal/hubforge/template.go:356
    summary: The poisoned-template failure tells the reader to take CopyHub in the test that writes, which is wrong advice when the template is a registered one
    fix: In internal/hubforge/template.go change the requireIntact message at line 356 from "take CopyHub in the test that writes" to "take CopyHub or CopyRegistered in the test that writes".
---

### [MEDIUM] Skip refusal is untested

**Location:** internal/hubforge/template_test.go:161
**Issue:** `templateBuildTB.Skip`, `Skipf` and `SkipNow` (template.go:297-307) refuse a skip, so the build's goroutine never ends without a recorded failure. The refusal is a stated behaviour of this diff, in the `Registration.Build` comment, doc.go, the PATTERN background and the summary. No `failingBuilds` row calls a skip method, so a regression that lets a skip through would pass every test. The embedded TB would then mark the first caller skipped from another goroutine, or leave `templateFor` with a nil template.

### [LOW] Non-fatal failure recording is untested

**Location:** internal/hubforge/template_test.go:161
**Issue:** `Error`, `Errorf` and `Fail` record a failure and let the build continue, and `templateFor` then discards the template and marks the slot failed. Every row exercises only `Fatalf` or a refusal, which ends the goroutine. Forwarding `Errorf` to the calling test would pass the table.

### [LOW] Build comment and package doc omit the once-only-registration clause

**Location:** internal/hubforge/template.go:73
**Issue:** D2's bound and the PATTERN background both say a build reaches no code whose result a once-only registration changes, such as a Bolt replay. The `Registration.Build` comment lists only the five process-wide calls, and doc.go's matching paragraph does the same. A build author reading the doc comment is not told the rule the review checks.

### [NIT] Poisoned-template message names only CopyHub

**Location:** internal/hubforge/template.go:356
**Issue:** `requireIntact` runs for registered templates through `CopyRegistered` too, and its advice to "take CopyHub" does not apply to a registered key.

## Rejected

- Lens code-tests, shape outside `Shapes` has no test: the check and its test gap predate the diff, which changes only the message wording; no new behaviour.
- Lens code-tests, a registration key equal to a shape name has no row: the mechanism is the identity check the shared-key test pins, and the summary's claim is borne out by `slotFor`.
- Lens code-correctness, a panicking build is not marked failed: the panic aborts the whole test binary, so no later caller exists to be misled; it is as loud as a panic in any test.
- Lens code-correctness, a build that leaves its goroutine with no recorded failure yields a nil template: every route through the TB records first, and only a direct `runtime.Goexit` in a build bypasses it, which is internal misuse.
- Handler, the refusing TB also refuses skips though D2 names five calls: the refusal keeps the build goroutine from ending without a recorded failure, and the PATTERN background declares it; it licenses no defect.
- Handler, D3's pack step is absent from the build order: the summary places the pack in the later `hubforge-slim-copy` task, so the fingerprint follows the tip check as the diff shows.
- Handler, declared PATTERN change: the overview line already carries the declared text, and the background diff adds the registered template and the build-function clause, so the declared change has landed.
