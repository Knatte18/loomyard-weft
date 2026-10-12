---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/10-gate-test-watchdog.md:6
    summary: Card 10 bundles gateslot's public, cross-package WatchdogLine, FindWatchdogQuit and BusyQuitWayForward into the gatecli watchdog card, though cards 11 to 13 consume them from three other packages
    fix: Run `lyx loom insert-card 10` to add a card gateslot-watchdog-line ahead of the watchdog card. The new card creates `internal/gateslot/watchdogline.go` and `plan:internal/gateslot#WatchdogLine`, `plan:internal/gateslot#FindWatchdogQuit` and `plan:internal/gateslot#BusyQuitWayForward` with the signatures card 10 gives them now. It moves card 10's Intent paragraph on `watchdogline.go` into its own Intent, and adds one table test `plan:internal/gateslot#TestWatchdogLine` that pins the render/find round trip, a line amid other output, and a log with no line, with `cmd/lyx/testdata/test-budget.yaml` under Edit for gateslot's row. In the renumbered watchdog card, drop the three gateslot Create targets and `internal/gateslot/watchdogline.go`, list the three symbols under Uses, and keep TestWatchdog's assertion that the buffer equals `gateslot.WatchdogLine`.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/11-webster-hang-gate.md:33
    summary: Card 11 adds the skip-rerun-after-hang edge but its Intent never states what a false match lets through or what bounds it, the bound D5 sets
    fix: In card 11's Intent, after the paragraph ending "an ordinary non-zero exit below 128 are no hang.", add a Bound paragraph saying that the markers are matched as text anywhere in the log, so a test that prints one, or a test binary the OOM killer ends, also reads as a hang. Any exit code of 128 or above reads as a hang whatever caused it, such as git's fatal 128 or a tool's 255 in a hub's verify command. All a match can skip is the one rerun, and a flaky failure in the same run reaches the fixer without its retry. Have the card's `websterengine` package-doc sentence state that bound beside D5's hang set.
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/02-state-tolerant-read.md:26
    summary: Card 2's list of comments the change makes wrong misses three that still describe an unknown field as undecodable or the poison rig as an unknown-field shape
    fix: In card 2 add these Edit targets, `internal/loomengine#CheckSeed` after `internal/loomengine#VerifySeedOwnership`, `internal/loomcli#poisonStatusFileMalformed` after `internal/loomcli#poisonStatusFile`, and `internal/loomshed#TestSeed_RefusesUndecodableFileAsExists` after `internal/loomshed#Seed`, and extend the list of comments the change makes wrong. CheckSeed's inline comment on its decode-failure branch, "A decode failure (malformed JSON or an unknown field)", names malformed JSON alone, and its doc's "step 1's identical strict read" drops "strict". poisonStatusFileMalformed's doc, "unlike the unknown-field shape poisonStatusFile writes", contrasts with the unrecognised-state shape instead. TestSeed_RefusesUndecodableFileAsExists's doc drops "or an unknown top-level field" from what cannot be decoded, and its row named "unknown top-level field" is renamed to say it is a decodable file Seed still refuses as existing. Checked with validate-plan --overlay against the {} baseline, with no new finding.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/04-state-rename.md:16
    summary: Card 4's per-file replace_all on ReadJSONStrict turns TestReadJSONStrict_MissingFile_NoMkdirAll into TestReadJSONNoMkdir_MissingFile_NoMkdirAll, not the TestReadJSONNoMkdir_MissingFileCreatesNoDirectory its own Rename row names, and the card fixes only the comment that names it by hand
    fix: In card 4's Intent, before the sentence on replace_all, add that in `internal/state/strict_test.go` the test `TestReadJSONStrict_MissingFile_NoMkdirAll` is first renamed by hand to `TestReadJSONNoMkdir_MissingFileCreatesNoDirectory`, its declaration and every mention, and only then does replace_all run on the file. Keep the existing sentence on the content-field comment.
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/11-webster-hang-gate.md:40
    summary: D5 counts res.TimedOut as a hang and D6 says the findings and stuck reason of a hang say the step was not re-run, but card 11 keeps failTimedOut and the timeout row of the stuck-reason test exactly as they are, so a timed-out step never says so
    fix: In card 11, keep `failTimedOut` as the path but have `renderVerifyGateFindings`'s timed-out sentence and `verifyGateStuckReason`'s timed-out case add that the step was not re-run. Replace "a table keeping its exact timeout row" in the TestVerifyGateStuckReason_HungReportNamesTheHang bullet with a timeout row that also wants "not re-run", and add a timed-out report row to the TestRenderVerifyGateFindings bullet asserting "not re-run".
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/10-gate-test-watchdog.md:57
    summary: Card 10's bound says a hand-set enclosing variable is ignored, but watchdogApplies honors any value naming a live ancestor, so a hand-set value naming the invoking shell, or pid 1 that card 9's IsAncestor reports as every process's ancestor, turns the watchdog off
    fix: In card 10's watchdogApplies bullet, replace "An empty, unparsable or non-ancestor value is ignored, such as one a tmux server a test started carried into its panes after leaving the outer run's tree, or one set by hand." with a statement that an empty, unparsable or non-ancestor value is ignored, such as one a tmux server a test started carried into its panes after leaving the outer run's tree. Follow it with a sentence that any value naming a live ancestor skips the watchdog, a hand-set one naming the invoking shell or pid 1 included. Then add "a run whose inherited variable names a live ancestor" to the Bound paragraph's list of runs bounded only by go test's own -timeout.
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/plan/10-gate-test-watchdog.md:54
    summary: Card 10 prescribes a branch where a sampler failure other than ErrUnsupported is logged and the next period samples again, but no TestWatchdog row reaches it
    fix: In card 10's TestWatchdog row list, add a row whose scripted sampler returns a plain error on its first call and then busy samples of a `.test` process for the whole window. The row asserts that run keeps sampling, signals that process exactly once and logs the sampler failure in the `logcapture.CaptureVerbose` buffer.
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/plan/08-gate-busy-quit-key.md:23
    summary: Card 8 says the negative busy_quit_sec refusal uses the wording of decodeConfig's other checks, yet prescribes want 0 or more where those say want at least 1
    fix: In card 8, change the refusal text in the decodeConfig bullet and in the TestLoadConfig row's expected error from `want 0 or more` to `want at least 0`, leaving the rest of each message as the card states it.
---

### [MEDIUM] gateslot's watchdog line symbols ride inside the gatecli watchdog card

**Location:** _lyx/plan/10-gate-test-watchdog.md:6

**Issue:**
The rubric's granularity rule gives a symbol of a module's public surface, or one reused across packages, its own card.
`WatchdogLine`, `FindWatchdogQuit` and `BusyQuitWayForward` are exported from `internal/gateslot`, and the Shared Decisions make them the sole render-and-find seam.
`internal/gatecli` writes the line, and cards 11, 12 and 13 read it from `internal/websterengine`, `internal/loomshed` and `internal/landingshed`.
Card 10 nonetheless creates them beside the unexported watchdog loop, its wiring, two unit tests, an integration scenario and four doc edits, so the shared seam has no reviewable unit of its own, and the only test reaching `FindWatchdogQuit` alone is a gatecli row.

### [MEDIUM] The rerun skip carries no stated bound

**Location:** _lyx/plan/11-webster-hang-gate.md:33

**Issue:**
Card 11 adds a routing edge that skips the gate's flake rerun whenever `classifyHang` matches.
D5 states what that edge lets through: a text match anywhere in the log, a test printing a marker, an OOM kill, and any exit code of 128 or above, git's 128 and a tool's 255 included, with only the one rerun skipped.
The card's Intent lists the match set and its order but never says what a false match costs or what bounds it, and the package-doc sentence it prescribes names only "D5's hang set and D6's report".
The attack-surface rule requires the introducing card to state both.

### [LOW] Card 2 misses comments that D1 makes wrong

**Location:** _lyx/plan/02-state-tolerant-read.md:26

**Issue:**
After card 2 no status read refuses an unknown key, and the poison rig switches to an unrecognised `state`.
Three comments outside its targets still say otherwise.
- `internal/loomengine/seed.go`'s inline comment in `CheckSeed`'s `rerr != nil` branch reads "A decode failure (malformed JSON or an unknown field)", and CheckSeed's doc calls step 1's read "identical strict read".
- `internal/loomcli/smoke_test.go`'s `poisonStatusFileMalformed` doc contrasts malformed JSON with "the unknown-field shape poisonStatusFile writes".
- `internal/loomshed/seed_test.go`'s `TestSeed_RefusesUndecodableFileAsExists` doc names "an unknown top-level field" as undecodable, and so does its row of that name.

Card 4's rewording of "strict" is limited to lines its rename touches, so it does not reach the first two.

### [LOW] Card 4's replace_all contradicts its own Rename target

**Location:** _lyx/plan/04-state-rename.md:16

**Issue:**
The Rename row maps `internal/state#TestReadJSONStrict_MissingFile_NoMkdirAll` to `TestReadJSONNoMkdir_MissingFileCreatesNoDirectory`.
The mechanic the card prescribes is one `replace_all` of `ReadJSONStrict` per file, which in `internal/state/strict_test.go` rewrites that declaration to `TestReadJSONNoMkdir_MissingFile_NoMkdirAll`.
The card sees this hazard for the `content` field comment and retargets it by hand, but not for the declaration itself, so following the card literally leaves a test whose name matches neither the Rename row nor the retargeted comment.

### [LOW] A timed-out step never says it was not re-run

**Location:** _lyx/plan/11-webster-hang-gate.md:40

**Issue:**
D5 lists `res.TimedOut` first among hangs, and D6 says that for a hung run the fixer's findings and the stuck reason say the step was not re-run.
`renderVerifyGateFindings` prints "The verify command did not finish within %s and was killed." for a timed-out report, and `verifyGateStuckReason`'s `report.TimedOut` case says the same.
Card 11 leaves `failTimedOut` unchanged and keeps the stuck-reason test's "exact timeout row", so the one hang the gate already knew is the one hang whose report does not say it was not re-run.

### [LOW] The enclosing-variable bound overstates what is ignored

**Location:** _lyx/plan/10-gate-test-watchdog.md:57

**Issue:**
`watchdogApplies` skips the watchdog whenever `enclosingGateEnv` parses as a pid that `isAncestor` reports as an ancestor.
Card 9's `IsAncestor` walks every parent pid from `os.Getppid()` upward and excludes only pid 0 and dead pids, so pid 1 and the invoking shell are ancestors of every `lyx gate test`.
A hand-set value naming either turns the watchdog off, yet card 10 lists "one set by hand" among the values that are ignored.
The bound should name that case, not deny it.

### [LOW] The sampler-failure branch has no test

**Location:** _lyx/plan/10-gate-test-watchdog.md:54

**Issue:**
Card 10 prescribes two failure branches in `run`: `errors.ErrUnsupported` logs once and ends the loop, and any other sampler error is logged and the next period samples again.
`TestWatchdog` has a row for the first and none for the second, so nothing pins that a transient `/proc` read failure neither ends the watchdog nor resets a binary's busy window into a missed quit.

### [NIT] The negative-key refusal departs from the wording it claims to follow

**Location:** _lyx/plan/08-gate-busy-quit-key.md:23

**Issue:**
`decodeConfig`'s other checks fail with `gate config key %q: %d; want at least 1`.
Card 8 says the new refusal is "in the wording of its other checks", yet prescribes `want 0 or more`, both in the decodeConfig bullet and in the TestLoadConfig row's expected error.

## Rejected

- Fidelity: D1 to D9, the added board-field-drop decision and each Constraint are carried, by cards 2 and 4 (D1), 3 (D2, D3), 5 and 6 (D4), 11 (D5, D6 webster), 12 and 13 (D6 loom and landing), 8, 9 and 10 (D7, D8, the daemon-wakeups, hub-wide-config, spawn-observability, test-speed and gate-slot constraints), 7 (D9) and 1 (board.json's encoding). No card adds scope the record does not license; card 7's four shortened PATTERN entries pay for D9's line under the headroom line, which D9 requires, and its byte counts (285 cut, 178 added) match `PATTERN.md`.
- Card order: card 2 changes `ReadJSONStrict` before card 4 renames it, card 3 extends `TestRun_UnknownTopLevelKeyBeforeRead` after card 2 flips it, card 8 adds `Limits.BusyQuit` before card 10 reads it, card 9 creates `proc.TreeCPU` and `proc.IsAncestor` before card 10 uses them, and card 10 creates the three gateslot symbols before cards 11 to 13 use them. No card uses a symbol a later card creates or renames.
- Member glyphs and re-sign arrows: every Edit card that names symbols lists them as member glyphs. Card 10 re-signs `runTiedChild` and `newCommand` with arrows. No other card changes a parameter, result or receiver of an existing member: `writeJSONUnlocked`, `verifyFailureFindings`, `verifyGate.verdict`, `newVerifyGate`, `renderVerifyGateFindings` and `verifyGateStuckReason` keep their signatures.
- Attack surface, busy_quit_sec: card 8 states that 0 turns the watchdog off, that the key is hub-wide only and not in `LocalKeys`, and that a negative value is refused; card 10 bounds -bench and -fuzz runs by go test's own -timeout.
- Reachability: the negative `busy_quit_sec` refusal has its TestLoadConfig row, and `LoadConfig` reaches `decodeConfig` after `configengine.Load` fills template keys, so no upstream check rejects -1. Every `classifyHang` branch has a first-run row, the rerun hang has its row, card 12's and card 13's watchdog branches have their rows, card 13's unreadable-log path is reached by the existing exact rows, and card 5's and card 6's tolerant decodes have their rows.
- Test economy: card 4 plans no test. Card 5's `TestSeed_UnknownKey` replaces the deleted `TestSeed_LegacyParentKey`, and card 6 adds a row to `TestCleanup`. Card 3 extends the two existing shedengine tests and deletes the one D2 reverses. No scan in `cmd/lyx` covers `DisallowUnknownFields`, so card 7's scan duplicates none. The budget rows match the counts `cmd/lyx/testbudget_test.go` takes, which exclude `TestMain`: gatecli 2 to 4, state 8 to 9, proc 17 to 18.
- Verify coverage: `## verify:` runs `go build`, vet under integration, tmux and llm, then `go test ./...` and `go test -tags integration ./...` in the template's order. The module has no nested module but `internal/planglyph/testdata/glyphchain`, which no card targets.
- Card 9's untagged `TestProcessTreeReads` spawns `sleep`, which `internal/proc`'s Test Tier Purity allowlist entry admits, as `tied_linux_test.go` and `isalive_test.go` already rely on.
