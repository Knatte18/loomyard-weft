```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. I checked everything outside the touched lines that reads, asserts or describes a behaviour the diff changes.

**Per change, the set found, covered or not**

- **`--gate card` runs the cards' own packages and plans no guard step** (`internal/gatecli/scope.go` `computeScope`):
  - Covered by the diff:
    - `doc.go` §scopes and §Guard step
    - the `--card` flag help (`cli.go:125`)
    - the `gate.yaml` template comment
    - the `loom-plan-spec.md` per-card and per-batch bullets
    - the `PATTERN-test-speed` background bullet
    - the Webster and Weaver stencils
    - the `govalues_test.go` file comment
    - the `TestGateTest_Scopes` rows
  - Already correct, no edit needed:
    - `cardDependents` in `carddependents_integration_test.go`: it calls the function directly, with no key.
    - `loom-block-plan-format.md:19`: the asserting test joins the card's targets, so it still runs at the card gate.
    - The `PATTERN.md` index line for `PATTERN-test-speed`: it says nothing about guard steps.
    - `doc.go:227-235`, the vet step: the batch gate "finds both" still holds.
  - Left wrong:
    - The long help of `cli.go:89`.
    - The `scope.guardStep` field comment at `scope.go:76`.
- **The fixer prompt re-runs only the failed checks** (`gatefix-body-card.md`):
  - The `gate_call` marker is kept, so the rendering in `detachfixer.go:266` and the marker rule in the file's header comment still hold.
  - The prompt points at "the command the report names". The report built in `failurereport.go` names no command for a run, only label, outcome and log. That gap belongs to the correctness and holistic review, not this lens.
- **`gateslot.CardGateKey` is exported and the duplicate key names are removed**:
  - Callers in `runplan`, `weaverengine`, `webstercli`, `priority.go`, `config.go` and `goReadGates` are all covered.
  - No gate-key `"card"` literal is left. The remaining `"card"` strings are flag names, JSON tags or the scope name.
  - The Weaver gate entry names (`card &lt;id&gt;`) are unchanged in value.
- **The Webster guard-fix rule moves to the batch gate section**:
  - The step 2 sanctioned-deviation sentence is updated.
  - Step 6's ` (fix)` form matches.
  - The self-fix and status rules are unchanged and still consistent.
- **Commit messages and departures**: no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailers, and none is needed. The directive declares no PATTERN changes, so "Declared PATTERN changes landed" raises nothing.

**Findings**

1. LOW, consistency, `internal/gatecli/cli.go:89`
   - The long help says a narrower scope under the go scoper runs the `//lym:guard` tests outside the scope, and that `--card` adds the tier-skipped ones. It does not name the `--gate card` exception, so it contradicts the new behaviour and the flag help at `cli.go:125`.
   - Evidence: `cli.go:89`; `scope.go:257` (`guardStep: !ownOnly`).
2. LOW, consistency, `internal/gatecli/scope.go:76`
   - The `guardStep` field comment still says the guard step is planned for "a computed, --base or --card scope under the go scoper". A `--gate card` scope no longer plans one.
   - Evidence: `scope.go:76` against `scope.go:245-257`.

**Tool calls**

1. A grep of `docs/`, `contracts/`, `pattern/` and `PATTERN.md` for card-gate claims about dependents or guards, plus the call in `carddependents_integration_test.go`: the manifest did not cover docs outside the diff.
2. A grep of the `websterengine`, `weaverengine`, `runplan` and stencils tests for the old wording, and of the whole repo for "own card gate"-style phrases: to find assertions and prose that pin the old behaviour.
3. A read of `loom-block-plan-format.md:12-22`, the `carddependents` test body and the `PATTERN.md` index line: to judge the hits from call 2.
