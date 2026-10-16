---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/05-carried-log-entries.md
    summary: card 5 says a carried record of an unknown of adds no log entry, but lists pool.applyCarried under Uses, so apply has no way to learn whether the carried record was applied
    fix: In 05-carried-log-entries.md, delete the line "- `internal/creelengine#pool.applyCarried`" from **Uses:** and add "- `internal/creelengine#pool.applyCarried` -> `func (p *pool) applyCarried(rec record, day string) bool`" to **Edit:** right after the `internal/creelengine#pool.apply` line. In the Intent, after "`pool.apply` adds one event-list entry for a carried record after `applyCarried` folds it", add that `applyCarried` reports true when it folded a held or posted record and false for an `of` it does not know, and that `apply` adds the entry only on true. This fix passes `lyx loom validate-plan --overlay` with no finding beyond the baseline.
---

### [LOW] Card 5 leaves the unknown-`of` case with no owner

**Location:** `_lyx/plan/05-carried-log-entries.md`, the Edit and Uses lists and the Intent paragraph on `pool.apply`.

**Issue:** The Intent says `pool.apply` adds a `carried` entry after `applyCarried` folds a record, and that a carried record whose `of` this binary does not know "adds none".
In `internal/creelengine/fold.go` `applyCarried` returns nothing, and its `default` branch only logs at Debug; the card lists it under **Uses**, so it stays unchanged.
Whoever implements the card must therefore copy the held/posted classification of `applyCarried` into `apply`, or break the card's own boundary by editing an unlisted function.
The copy drifts the next time `applyCarried` learns a new `of`, and the `TestJournal_Fold` row the card extends would then pass while the live classification disagrees.
Making `applyCarried` report whether it applied keeps the classification in one place and fits the decision record's D5, which wants each carried record to add one entry.

## Rejected

- The plan's `## verify:` compiles the `tmux` tier and does not run it, although card 11 changes `smokeHub.addStrand`, which the tmux smoke test shares: `pattern/PATTERN-test-speed.md` assigns that tier to Publish's `publish_verify`, so the plan's `go vet -tags tmux ./...` is the form the pattern prescribes, and the constraint wins over the rubric's wording.
- Card 2 repeats the setups of the existing "as EXTERNAL the posting and waiting verbs report their outcomes" row in a new row: the new row pins the stderr fallback of the same envelopes, which no existing row covers, so it is a new behavior and not a duplicate.
- Card 4's reap row starts `sh` in `internal/creelcli/cli_test.go`: that file is `integration`-tagged, so `PATTERN-test-speed`'s untagged-spawn rule does not apply.
- Card 8 describes its `TestScheduler_Injects` row as moving from one two-minute advance to probe-by-probe stepping while card 3 also changes rows that read a stall and expire: the row asserts no answer stall today, so card 3 leaves it alone and the two cards do not overlap.
- Card 10's `TestRunner_AcceptStartupGate` is a new top-level test: it covers a new exported method no existing test reaches, and the card raises the `internal/shuttleengine` row of `test-budget.yaml` by the one test.
- `claudeengine.Startup` classifies both the folder-trust gate and the bypass-permissions gate as `StartupTrustPrompt`, and `TrustDismissSequence` accepts both, so card 10's seam covers the two gates D9 names with no claude-specific code.
