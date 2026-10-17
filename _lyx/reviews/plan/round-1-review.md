---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/01-log-id-reminded.md (Edit list, ImpactSummary, and the paragraph on existing LogFilter{ID} assertions)
    summary: Card 1 misses the creelcli integration subtest log --id prints a carried entry's kind, which asserts log --id 1 lists exactly one entry and after D4 also sees the reminded event the daemon's scheduler journals for the carried unread message
    fix: >-
      In 01-log-id-reminded.md add `internal/creelcli#TestCreelCLI` to the Edit list after `internal/creelengine_test#TestScheduler_Reminders`. In the Intent, add a sentence that the subtest `log --id prints a carried entry's kind` in `internal/creelcli/cli_test.go` must change, since its carried unread message #1 to the idle LiveStrand beta is reminded once the daemon starts, so `log --id 1` now lists `carried` and `reminded`. Prescribe one of two fixes. Option one gives beta `creelfake.DormantStrand(beta)` in its `SetStrands` call, so no reminder is typed and the one-entry assertion stays. Option two asserts that the first entry is the `carried` one with kind mail, and drops the exact count. Correct the checked-assertions paragraph so it names this creelcli subtest. Correct the ImpactSummary so it says one existing creelcli assertion changes.
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/plan/00-overview.md (## verify:)
    summary: Card 1 targets tools/sandbox/SANDBOX-CREEL-SUITE.md, which tools/sandbox/suite.go embeds and whose suite_test.go asserts the embedded body, but the verify section never runs the tools/sandbox package's tests
    fix: >-
      In 00-overview.md's `## verify:` block, change the line `lyx gate test ./internal/creelengine ./internal/creelcli` to `lyx gate test ./internal/creelengine ./internal/creelcli ./tools/sandbox`. The sandbox tests carry no build tag, so the integration line stays as it is.
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/02-refusal-prefix-once.md (Test, first row)
    summary: The first row quotes the engine refusal as creel unknown strand, but checkReceiverLocked's text is creel unknown receiver with the names that exist
    fix: >-
      In 02-refusal-prefix-once.md's first test row, replace `refused as an unknown strand (`creel: unknown strand ...`)` with `refused as an unknown receiver (`creel: unknown receiver "..."; the names that exist: ...`)`.
---

### [BLOCKING] Card 1 leaves a creelcli `log --id` assertion that D4 breaks

**Location:** `_lyx/plan/01-log-id-reminded.md`. The evidence is in `internal/creelcli/cli_test.go:490-509`.
**Issue:** The subtest `log --id prints a carried entry's kind` writes a journal holding a carried, unread message #1 for `beta`.
It registers `beta` as `creelfake.LiveStrand`, which the fake reports idle when it is unscripted (`internal/testkit/creelfake` `Idle`).
It then starts the daemon and asserts `len(entries) != 1` on `log --id 1`.
`creelengine.Run` starts the scheduler (`internal/creelengine/server.go:88`), and a worker probes before its first wait (`internal/creelengine/deliver.go:126`, `:143`).
So `beta` is reminded of #1 at startup with `Unread: [1]` (`deliver.go:553-556`).
Under D4's `Unread` clause, `log --id 1` then lists `carried` and `reminded`.
Depending on whether the probe runs before the `log` call, the plan's own `--tags integration ./internal/creelcli` verify either fails or flakes.
Card 1's sweep of existing `LogFilter{ID}` assertions covered only creelengine's tests, and its ImpactSummary states that no existing assertion changes.
The decision record's first Open risk asks the plan to check each such assertion.
The fix was overlaid with `lyx loom validate-plan --overlay` together with F2, and the run reported no finding.

### [MEDIUM] The verify section does not run the tests of `tools/sandbox`, which card 1 targets

**Location:** `_lyx/plan/00-overview.md` `## verify:`
**Issue:** `SANDBOX-CREEL-SUITE.md` is a file of the `tools/sandbox` package of the root module, and `tools/sandbox/suite.go` embeds it with `//go:embed SANDBOX-CREEL-SUITE.md`.
`tools/sandbox/suite_test.go` asserts that each scheme file holds the embedded suite body.
The verify section builds `./cmd/lyx` and runs `./internal/creelengine` and `./internal/creelcli` only.
`./cmd/lyx` does not import `tools/sandbox`, so nothing in verify builds or tests the package card 1 edits.
The writer stencil requires the section to cover every package a card targets.
The fix was overlaid with F1, and the run reported no finding.

### [NIT] Card 2 misquotes the unknown-receiver refusal

**Location:** `_lyx/plan/02-refusal-prefix-once.md`, test row 1.
**Issue:** `checkReceiverLocked` returns `creel: unknown receiver %q; the names that exist: ...` (`internal/creelengine/pool.go:205`), not `creel: unknown strand ...`.
The input reaches the refusal and the row asserts only the prefix count, so this affects only an implementer who copies the quoted text into an assertion.

## Rejected

- way-forward, LOW: a `post --wait-reply` whose reply arrived but whose envelope cannot be written names only the posted id, and the reply body cannot be recovered. The decision record's Open risks accepts this bound in so many words, so the plan follows its answer key and no plan change is licensed.

## Holistic notes

- Fidelity: D1–D5 and every Constraint have a carrying card.
  Every acceptance-criteria behaviour has a planned row or an existing test.
  `read`'s behaviour on a refusing writer is the one covered by an existing test (`cli_test.go:568`).
  No card adds scope the record does not license.
  Card 1's `logCmd` help rewording follows from D4's behaviour change under the docs-in-same-commit rule.
- Re-sign arrow: `writeRefusal` carries one for its new `done` parameter.
  `refuse` and `runVerb` keep their signatures, and `result` gains a struct field, which takes no arrow.
- Granularity: `partialWriter` and its two methods sit on the one card that uses them, which matches the Granularity bullet.
- Test economy: card 1's four `Log` assertions each reach a distinct filter clause.
  Card 2's and card 3's new rows each pin a fallback or lost-answer behaviour that no existing `TestCreelCLI` subtest asserts.
  The existing standalone-repository and argument-count subtests run into a writable stdout.
- Card order: card 3 re-signs `writeRefusal` on top of card 2's condition, and its Intent names the line card 2 left. No card depends on a later one.
- Test budget: no card adds a top-level test.
