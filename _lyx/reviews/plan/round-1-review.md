---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/18-test-socket-keys.md
    summary: Card 18 names five files but leaves the other raw kill-server test sites to an open-ended survey result, though a grep finds them now
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/17-tmuxkit-socket-cleanup.md
    summary: KillOnCleanup states no bound on its removal; with TMUX_TMPDIR unset it reaches the real per-user socket directory, and it removes after kill-server ran rather than after no server answers
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/00-overview.md
    summary: Shared Decision says no card before 9 adds a test file, but card 1 creates main_test.go and card 8 creates testmain_test.go
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/20-testtiming-redundancy.md
    summary: Card 20 recomputes per-package count, wall and serial time without naming reuse of card 1's aggregation, so D1's single-parser rationale can be lost
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/plan/10-pattern-testing-consolidation.md
    summary: Card 10 says tiersleep_test.go reports a retired ID, but the file names none and is not a target
---

### [MEDIUM] Card 18 leaves known socket-leak sources to a survey

**Location:** `_lyx/plan/18-test-socket-keys.md`
**Issue:** D9 names the reedengine contract keys and the reedcli attach and dotfill harnesses, and card 18 targets exactly those.
Its further sources are "every further source card 17's survey named", added to the commit with no target.
A grep of the tests for `kill-server` shows the sites now: `internal/standalonegeom/naming_integration_test.go` (a hand-formed `"lyx-"+hash8` key killed by a raw `exec.Command`), `internal/reedcli/naming_integration_test.go`, `internal/reedcli/smoke_coldstart_test.go`, `smoke_debuglog_test.go`, `smoke_resume_test.go`, `smoke_teardown_test.go`, `internal/reedengine/ensuresession_integration_test.go` and `mouse_boot_integration_test.go`.
The decision record's notes say `lyx-001-*`, `lyx-contract-*` and `lyx-Test*` still escape, so these are the expected sources.
**Fix:** name those files in card 18's `Edit` list and keep the survey clause for sources the grep misses.

### [LOW] KillOnCleanup states no bound

**Location:** `_lyx/plan/17-tmuxkit-socket-cleanup.md`
**Issue:** `KillOnCleanup` removes a socket file under "the current `TMUX_TMPDIR`'s per-user directory".
With `TMUX_TMPDIR` unset that is the operator's real `/tmp/tmux-$UID/`.
It removes "after `kill-server` has run", not after no server answers, so a failed kill would unlink a live server's socket.
The Intent does not say that it removes one path only and never globs.
**Fix:** add a bound sentence: it removes only the one path for its key, never globs or sweeps, and only when the path is a socket and no server answers.

### [LOW] Overview's "no test file added before card 9" is false

**Location:** `_lyx/plan/00-overview.md`
**Issue:** Card 1 creates `cmd/testtiming/main_test.go` and card 8 creates `internal/gitrepo/internal/gitoracle/testmain_test.go`, both before card 9.
Both are untagged, so neither moves a file in or out of the `integration` tag (D4), but card 1's test runs inside the integration run that cards 2 and 9 both measure.
**Fix:** reword the Shared Decision to what D4 needs: no card before 9 adds, removes or retags a tagged test file; untagged additions in cards 1 and 8 are in both measured runs.

### [LOW] Card 20 does not reuse card 1's aggregation

**Location:** `_lyx/plan/20-testtiming-redundancy.md`
**Issue:** D1 picks `cmd/testtiming` so the event parsing is not duplicated.
Card 20 derives test count, wall time and serial time for each package, and has no `Uses` of card 1's `pkgResult` and `parseLine`.
**Fix:** add them under `Uses` and say the report header takes its numbers from that aggregation, run over the same `go test -json` stream.

### [NIT] Card 10 names a file that has no retired ID

**Location:** `_lyx/plan/10-pattern-testing-consolidation.md`
**Issue:** The card says `cmd/lyx/tiersleep_test.go` reports `PATTERN-test-speed`.
The file names no entry ID, only the prose "Test Tier Purity Invariant", and is not in the target list.
**Fix:** drop it from that sentence; `tierpurity_test.go` carries the report.

## Verdicts on the focus questions

- Card 19 states every D9 bound (key, teardown, socket, no answering server, never a glob, identity re-check); card 17 does not (F2).
- Cards 11 and 15 state the guard's static-shape bound and the `exec`-identifier check on `llmkit` files.
- Cards 13 and 14 against verify: every moved package is run by `-tags tmux` or compiled by `go vet -tags llm`; no file is left under an unchecked tag.
- Card 10 against D8: a grep of the five retired IDs outside `_lyx/` returns 41 files, all in card 10's list; the backgrounds are deleted and created as D8 says.
- Cards 3-9 against D4 and D5: no retag before card 9 and every seam is per instance; only the overview wording is off (F3).
- Cards 1, 20, 21 against D1-D3: card 20 carries every D2 rule; no duplicated tests, apart from F4.
- Granularity: cards 13 and 20 are large bundles but their units share files and tests, so no finding.
- Coverage of D1-D10 and the after-Discussion constraint is complete.
  Card 16's `crucible/` and `claims_test.go` edits follow from retiring `smoke` and no card touches `cardGateCommand`.
