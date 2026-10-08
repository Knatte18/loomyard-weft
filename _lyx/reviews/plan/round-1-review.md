---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: decision
    location: _lyx/plan/02-record-batch-resolves-head-sha.md (Intent, resolver bullets)
    summary: resolveReportHead resolves only lowercase hex, so an uppercase abbreviation of HEAD meets the mismatch refusal although D1 resolves any 4-to-39-character hex string and git disambiguates uppercase
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/05-derived-overview-frame.md (ImpactSummary)
    summary: ImpactSummary says five plan-edit helpers take the webster dir, but overviewIndexOnly and overviewRefusal take the recorded frame instead; only three gain websterDir
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/01-pattern-fill-directive.md (Intent, second paragraph)
    summary: FillDirective is told to wrap a fill error as Directive wraps it today, but today's wrap names the stencil and FillDirective takes no stencil name
---

### [LOW] An uppercase hex abbreviation is left unresolved, against D1

**Location:** `_lyx/plan/02-record-batch-resolves-head-sha.md`, the `resolveReportHead` bullets ("a lowercase hex value of 4 to 39 digits is passed to `git.CommitsNamedBy`"; "any other value (not hex, or longer than 40) returns unchanged").
**Issue:** D1 says "A hex string of 4 to 39 characters is resolved to the one commit it abbreviates" and "A value that is not hex is left as given".
The card narrows resolution to lowercase hex, so an uppercase hex abbreviation, which is hex, falls into the "returns unchanged" bucket and meets the mismatch refusal in record-batch and recover-batch and the missing-commit path in reset, forcing the re-drive D1 exists to remove.
git itself resolves it: in this repository `git rev-parse --disambiguate=7B3D140C` prints `7b3d140cb3f386f3c3c61e10e2b0eba7d3530d72` with exit 0, the same as the lowercase form.
The probe also confirmed the card's open question: a prefix naming nothing (`--disambiguate=ffffffffff`) prints nothing and exits 0, so an empty answer is no match, not an error.
**Fix:** accept hex of either case for the 4-to-39-digit branch (lowercasing the value before the probe, or passing it as given since git accepts both), and state in the Intent that a resolved value is the lowercase full SHA `CommitsNamedBy` returns; keep "not hex" meaning a value with a non-hex character.

### [LOW] Card 5's ImpactSummary miscounts the helpers that take the webster dir

**Location:** `_lyx/plan/05-derived-overview-frame.md`, `**ImpactSummary:**`.
**Issue:** the summary says "five unexported or exported plan-edit helpers take the webster dir".
The card's own re-sign arrows disagree: `overviewIndexOnly(recorded, planDir string)` and `overviewRefusal(recorded string)` take the recorded frame, not websterDir;
only `fingerprintMismatchWayForward`, `PlanEditError` and `batchCardEditError` gain websterDir.
**Fix:** reword to say three helpers gain the webster dir and two take the recorded frame in place of the state.

### [NIT] FillDirective's error-wrap instruction cannot be followed literally

**Location:** `_lyx/plan/01-pattern-fill-directive.md`, Intent, "wrapping a fill error as `Directive` wraps it today".
**Issue:** `Directive` wraps a fill error as `pattern: fill directive stencil %q: %w` with the stencil's name (`internal/pattern/pattern.go:129`).
`FillDirective(stencilText, overview string)` has no name to put there, so either the wrap drops the name, changing `Directive`'s error text, or `Directive` must re-wrap with the name, which the card does not say.
**Fix:** say which: `FillDirective` wraps as `pattern: fill directive stencil: %w`, and `Directive` keeps its named wrap around `FillDirective`'s error; or state that `Directive`'s fill-error text loses the stencil name.

## Verdicts on the focus entries

- **Card 3 attack surface:** not a defect.
  Card 3 adds no new mechanism; it routes two sites through card 2's resolver, whose Intent states the bound (object names only, exactly one commit).
  Card 3's own Intent names what bounds the accepted value at each site: in recover-batch the resolved digest head goes through `reconcileReportHead` (`recoverbatch.go` runs it after `refuseMidMerge`), and in reset "a resolved value then meets the descends-from-start check as a full SHA does".
  Today `reportHeadSHA` already passes an abbreviation to `SHAExists`, which resolves it through go-git's first-match `ResolveRevision` (`internal/gitrepo/gitrepo.go:220`), so the card tightens reset rather than relaxing it.
- **Card 4 re-read identity:** the rows exercise the drop, not only the happy path.
  The second-call row asserts "the finding the first call dispositioned not reported again", which reaches `isDispositioned` with a re-read finding.
  The claim holds against the code: `forkKey` keys by transcript, class and per-class ordinal (`audit.go:123`), and `ForkReport.WritePaths` is in transcript order (`internal/shuttleengine/forkaudit.go:45`), so a grown transcript keeps earlier keys.
- **Cards 2 and 4 ordering:** no defect.
  Both edit `RecordBatch` at disjoint points (card 2 after `ParseReport` and on the correctness path; card 4 at the seen set, the attribution count and the advance), and neither uses a symbol the other creates.
  A generic error such as D1's refusal is not persisted by the CLI (`internal/webstercli/recordbatch.go` saves only the fingerprint re-baseline), so a re-run after the fork fixes `head_sha` still sees the transcript as new.
- **Card 5 re-sign arrows:** all five re-signed helpers carry the arrow, and the caller list matches the code: `BeginBatch` (`beginbatch.go:321`), `RecordBatch` (`recordbatch.go:379`), `PersistRecoveryTerminal` (`recoverbatch.go:538`), `Run` (`runlevel.go:480`) and `validateCmd` (`validate.go:237`), plus `Rebaseline`.
  The callers' signatures do not change, so plain Edit members are right.
- **Card 7 acceptance and test economy:** the fixed-context section, the second-Merriam skip and the directive-wrapper subtraction each have a row in `TestCalibrate`, and the measurement point has its row in card 6's `TestCountRun`.
  `merriamTexts` has no test of its own; it is reached only through `Calibrate`.
- **Fidelity sweep:** every D1 to D5 sentence and every listed Constraint has a carrying card.
  Card 1's `FillDirective` export is the mechanism D4's "filled as `pattern.Directive` fills it" needs without duplicating the fill; D4's "no new export" sentence concerns the constant only.
  Card 2's `Git` method is licensed by D1 and the `PATTERN-gitexec-checked-call` constraint; `realGit` and `fakeGit` are its only implementers.
- **Verify coverage:** `## verify:` runs `go test ./...` and `go test -tags integration ./...`, which cover every targeted package, in the template's order (build, vet integration, vet tmux, vet llm, untagged, integration).
