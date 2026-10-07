---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/11-tmux-tier-green.md:3
    summary: Card 11 says the failure text surfaces through the restartSession and addStrandIn helpers, which live in other reedcli test files, but its target and stop rule cover only smoke_dotfill_test.go
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/11-tmux-tier-green.md:18
    summary: Card 11's stop branch has no effect on card 12, so a failing tmux tier would still be followed by the card that sets publish_verify, against D4's ordering
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/09-mark-guards.md:3
    summary: Card 9's Edit list names many files whose tests read only their own package directory, which the card's own criterion leaves unmarked, and the card does not say the list is a candidate set
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/11-tmux-tier-green.md:10
    summary: Card 11 pins a snapshot of the plan writer's own run (machine, kernel, load average, which commands passed) that goes stale and does not decide the implementer's branch
---

### [MEDIUM] Card 11's scope does not cover where its own premise says the failure surfaces

**Location:** `_lyx/plan/11-tmux-tier-green.md` (`**Edit:**`, and the stop rule in `**Intent:**`)
**Issue:** The card states that the "no space for new pane" text "surfaces through reed's own splits (`restartSession`, `addStrandIn`)".
Both helpers are test code in other files: `restartSession` is in `internal/reedcli/smoke_lifecycle_test.go` and `addStrandIn` is in `internal/reedcli/smoke_test.go`.
They call reed's production `up` and `add`, whose splits are where the message is produced (`internal/reedengine/selvagepane.go`, `doc.go`).
The card's only target is `smoke_dotfill_test.go`, and its stop rule sends "a cause outside that file" to a report-and-stop, so a cause in the shared test helpers, which is test-only and which D4's auto-mode assumption ("a fixable test-environment problem") covers, would halt the card.
The card also lists a target it expects to leave unchanged on the pass branch, without saying the target is a candidate.
I checked the premise against the tree: `-run 'TestSmokeDotFill$/CrossClientControl'` and the full `go test -tags tmux ./...` both pass on this clean tree, `04d9b57c0` is not an ancestor, and every `newDotFillHarnessOn` caller passes 140 by 42.
The pass branch is therefore right, and the cause site is correctly left to captured output; only the scope of the fix branch is wrong.
**Fix:** Name the `internal/reedcli` test-support files (`smoke_test.go`, `smoke_lifecycle_test.go`) as candidate targets next to `smoke_dotfill_test.go`, say all three are edited only where the captured output shows the cause, and keep reed production code as the one stop condition.

### [MEDIUM] Card 11's stop branch leaves card 12 free to set `publish_verify` over a failing tier

**Location:** `_lyx/plan/11-tmux-tier-green.md` (`**Intent:**`, the two-branch list), `_lyx/plan/12-publish-verify.md` (`**Intent:**`)
**Issue:** D4 requires that no commit of the task sets `publish_verify` while the tier fails, and its Open risks say that ordering is "not optional".
Card 11 says that when the cause is outside its target it "reports the file and the output and stops", and that card 12 "comes after this one".
Nothing says what the stop does to the run: card 12 has no `Uses` edge on card 11 and its Intent never mentions the precondition, so the sequencer runs it in declared order after a card that stopped.
The two cards disagree at the seam: card 11 promises an ordering that holds only on its pass and fix branches.
**Fix:** In card 11, make the stop branch end the card unfinished with the report, so the run halts at card 11 and card 12 is not started until the tier passes.
In card 12's Intent, state that it assumes card 11 left the `tmux` tier passing and sets the repository's value only after that card finished.

### [MEDIUM] Card 9's Edit list is neither derived from its criterion nor stated as a candidate set

**Location:** `_lyx/plan/09-mark-guards.md` (`**Edit:**`, and the targets paragraph of `**Intent:**`)
**Issue:** The criterion marks a test only when it reads repository files outside its own package directory, and leaves a test that reads only its own directory unmarked.
Many listed files hold only own-directory scans, so under the card's own rule they end unchanged: `internal/shedverbs/seam_enforcement_test.go`, `internal/cliwire/seam_enforcement_test.go`, `internal/shuttleengine/seam_enforcement_test.go`, `internal/shuttleengine/completionsignal_enforcement_test.go`, `internal/fabricengine/mergevocab_test.go`, `internal/gitrepo/noforceadd_test.go`, `internal/lyxcwd/raddle_guard_test.go`, `internal/planglyph/chokepoint_enforcement_test.go`, `internal/planglyph/status_enforcement_test.go`, `internal/planparser/shape_test.go`, `internal/reedengine/panebin_enforcement_test.go`, `internal/reedengine/selvagepane_enforcement_test.go`, `internal/shedcli/table_test.go`, `tools/sandbox/pathresolve_guard_test.go`, and `cmd/lyx/prerunlogging_test.go` (which reads only `cmd/lyx/main.go`).
`cmd/lyx/crosscompile_test.go` imports `scankit` only for `Root` and builds the module through the compiler, so whether it counts is left open.
Same-shaped own-directory scans are not listed, for example `internal/agentname/seam_enforcement_test.go` and `internal/loomshed/seam_enforcement_test.go`, so the list follows no stated rule.
The card never says an Edit target may end unchanged, so an implementer who reads the list as authoritative marks every listed file and widens the round gate's guard set against D5's definition.
**Fix:** State in the targets paragraph that the Edit list is the candidate set to check: a listed file none of whose tests reads outside its own package directory, with `crosscompile_test.go` named as one of them because it builds through the compiler rather than scanning, stays unchanged, and the criterion alone decides what is marked.

### [LOW] Card 11 pins a snapshot of the plan writer's run

**Location:** `_lyx/plan/11-tmux-tier-green.md` (second paragraph of `**Intent:**`)
**Issue:** The card records the machine class, kernel, tmux version, a load average "about 9" and the three runs that passed.
That is a measurement of one moment and does not change what the implementer does: the card has the implementer run the tier first either way.
**Fix:** Keep what decides the branch (the recorded revision is not an ancestor, the failing test is now the `CrossClientControl` step of `TestSmokeDotFill`, the harness window is sized explicitly) and drop the machine, load and run list.
