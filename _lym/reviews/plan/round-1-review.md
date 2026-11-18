---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:affected-set
    location: _lym/plan/03-engine-worktree-geometry.md:40
    summary: Card 3 misses the launching wiring battencli primeShuttleRunner, which beamcli's runnerTidyShuttle reaches through battencli.ShuttleRunner to launch the dobby's board-tidy agent in the beam worktree, so that launch is told no worktree and ignores the beams pin and the beam's entry (D4 every launch, D9).
    fix: In card 3, add `internal/battencli#primeShuttleRunner` under Edit, and add to the Intent that it switches from `shuttleengines.New(shuttleConfig, dirs)` to `shuttleengines.NewForWorktree(shuttleConfig, dirs, hubgeom.EngineWorktree(location))`, so the tidy agent `beamcli`'s `runnerTidyShuttle.Start` launches through `battencli.ShuttleRunner` runs under the beam's account; raise the ImpactSummary's count of launching members from eight to nine. Checked with `lym loom validate-plan --overlay`, no new blocking finding.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/14-in-run-relaunch.md:39
    summary: A usage limit on the skill-load turn never reaches Wait, since launch runs loadSkillsThenPrompt before Wait and readLoadTurn counts the limited turn end as an API error, retyping the load up to loadHoldMaxAPIErrors and ending OutcomeAPIError, so on that path no card records blocked_until (D5) or asks the switch (D7), and card 14's skill-load-turn row cannot be reached.
    fix: In card 14, add `internal/shuttleengine#Run.readLoadTurn` under Edit and state in the Intent that a load turn end the engine's `TurnEndActivity` reads as a usage limit goes to `switchAccountAtLimit` before the load hold counts it, so every hit records `blocked_until` through `AccountSwitcher.SwitchAccount`; on false the load hold goes on as today; on true the run relaunches under the new account. State the relaunch shape for this case in the Intent, a resume of the same session id that keeps the skill-load turn and the run's deferred prompt line, and have the parent record it with `lym loom decision add`, since D7 step 2 names only the continue line. Replace the row 'a limit hit during the skill-load turn keeps the deferred prompt line' in `TestWait_UsageLimitSwitchRelaunchesInTheRun` with a row driving a usage-limited load turn end through `Start`, asserting `blocked_until` is recorded and the relaunch runs. Checked with `lym loom validate-plan --overlay`, no new blocking finding.
  - id: F3
    severity: BLOCKING
    class: scope
    origin: lens:repo-invariants
    location: _lym/plan/02-claude-account-config.md:8
    summary: Card 2 classifies accounts and account_pins CategoryMachine, but configreg's TestModules_EveryTemplateKeyClassified pins every module's MachineKeys and wants an empty list for claude, so it fails once card 2 lands and no card lists it.
    fix: In card 2, add `internal/configreg#TestModules_EveryTemplateKeyClassified` under Edit, and add to the Intent that its `wantMachineKeys` gains a claude case wanting `[]string{"account_pins", "accounts"}`. Checked with `lym loom validate-plan --overlay`, no new blocking finding.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/16-refused-resume-starts-fresh.md:27
    summary: Card 16 decides a start-path refused resume in loadSkillsThenPrompt, but the load turn's first API-error turn end is consumed inside loadSkillTurn and readLoadTurn, which hold and retype, so loadSkillsThenPrompt sees only a passed load or the capped error after eight errors, and the row for a copied resume whose load turn ends on an API error cannot be reached at the first turn end.
    fix: In card 16, add `internal/shuttleengine#Run.readLoadTurn` under Edit and move the start-path case there, so that the first load turn end past `LaunchOffset` that is an API error other than a usage limit, on a record with `TranscriptCopied`, ends the load hold and returns to `loadSkillsThenPrompt`, which runs `relaunchUnderAccount(true)`. Keep the row in `TestWait_RefusedResumeStartsFresh`, asserting one load is typed before the fresh relaunch. Checked with `lym loom validate-plan --overlay`, no new blocking finding.
  - id: F5
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/05-account-pick-at-making.md:24
    summary: Card 5 adds adoptDeps.PickAccount and calls it after each successful AddPair, but the adopt tests' shared builder world.deps builds adoptDeps without it, so every existing adopt test that adds a pair calls a nil func and panics; card 5 lists only TestAdopt.
    fix: In card 5, add `internal/adoptcli#world.deps` under Edit and state in the Intent that it sets a `PickAccount` that records each slug, which the new `TestAdopt` rows assert. Checked with `lym loom validate-plan --overlay`, no new blocking finding.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/12-run-relaunching-record.md:14
    summary: Cards 7, 8, 11 and 12 list a doc.go under Extends while their Intent falsifies sentences already in it, and the done-when paragraph of the PATTERN-shuttle-provider-seam background goes stale too; the method treats every doc as add-only.
    fix: "Move each doc to Edit and name in the Intent the sentences it rewrites. Card 7: `internal/shuttleengine/claudeengine/doc.go` (the project directory under `<home>/.claude/projects`, the config directory and plugins file paths) and `internal/shuttleengines/doc.go` (RecordReaders and SessionNamers built with no config directories). Card 8: `internal/shuttleengine/claudeengine/doc.go` (the one session registry of the resume and liveness reads). Card 11: `internal/reedengine/doc.go` (Resume relaunching or dropping every non-live strand). Card 12: `internal/shuttleengine/doc.go` (the stop verb's removable strands, the held-session branch of a step stop, and the carry-over's waiter-gone rule). In card 14, which already edits `pattern/PATTERN-shuttle-provider-seam.md`, add to the Intent that its done-when paragraph gains the held-strand exception. Checked with `lym loom validate-plan --overlay`, no new blocking finding."
  - id: F7
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/plan/14-in-run-relaunch.md:48
    summary: Card 14's deferred-prompt rule types the record's unsent PromptLine in place of the continue line, but step 2 prepares the launch with the continue line as its prompt and SkipSkillLoadTurn set, which keeps the prompt on the launch line, so the line cannot replace it; the rule also departs from D7 step 2's empty PromptLine with no recorded decision.
    fix: In card 14, delete the sentence starting 'When the record holds a deferred `PromptLine` whose own turn never started' from step 2, and the matching row; the only case it serves is the load-turn limit, whose relaunch shape F2's fix settles through a decision the parent records.
  - id: F8
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/plan/11-reed-hold-and-relaunch-ops.md:38
    summary: Card 11's held mark makes reed's resume skip a strand, neither relaunching nor dropping it, and reconcile leave it, but its Intent states no bound on a hold whose holder died; only a stop, ReleaseStrandHold, RelaunchStrand or card 15's settle on a later attach clears it.
    fix: "In card 11, add to the Intent what the hold lets through and what bounds it: reed resume and reconcile skip a held strand for as long as it is held, so a Wait process that dies between step 1 and step 4 leaves a strand down; the hold ends at the stop verb's removal, `ReleaseStrandHold`, `RelaunchStrand`, or card 15's settle at the next attach or Wait past the relaunch bound, and nothing else clears it."
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/04-claude-account-store.md:21
    summary: Card 4's TestResolveAccount is a per-symbol test whose resolution rows (hub pin over entry and beams pin, beams pin on prime and beam only, a blocked beams pin yielding or standing, a blocked hub pin standing) card 6's public-surface TestPrepare_ExportsTheResolvedAccount repeats row for row.
    fix: In card 4, remove the `plan:internal/shuttleengine/claudeengine#TestResolveAccount` Create bullet and its rows sentence. In card 6, add its two rows no other test has to `TestPrepare_ExportsTheResolvedAccount`, an entry naming a removed or retired account launches with neither export nor unset, and an engine told no worktree resolves only the hub pin. Checked with `lym loom validate-plan --overlay`, no new blocking finding.
  - id: F10
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: _lym/plan/12-run-relaunching-record.md:23
    summary: D7 says a reader reporting a run's session state reports a relaunching run as relaunching; card 12 adds RunSessionState.Relaunching, but lym shuttle state builds each run's output from stateFields alone and no card prints the flag.
    fix: In card 12, add `internal/shuttlecli#shuttleCLI.stateCmd` under Edit, state that each run's map gains `relaunching` from `RunSessionState.Relaunching`, and add a row to the state verb's existing test asserting it. Checked with `lym loom validate-plan --overlay`, no new blocking finding.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/12-run-relaunching-record.md:27
    summary: Card 12 makes strandRemovable read a held strand as removable, which also changes removeSupersededStrands and StopStepRuns' untied branch, so a relaunching run that is a superseded attach candidate now has its held strand stopped; the card names neither site.
    fix: In card 12's `strandRemovable` bullet, name `removeSupersededStrands` and `StopStepRuns`' untied branch and state that each now reaches a held strand, removing it through `stopRecorded` for a superseded candidate and reporting it untied, as intended for a relaunch whose run a fresh run supersedes.
  - id: F12
    severity: LOW
    class: scope
    origin: lens:guard-reach
    location: _lym/plan/05-account-pick-at-making.md:44
    summary: Card 5 picks only after top.Add succeeds, so a Worktree-Create row re-entered after a crash between Add and the pick takes the complete branch, which reconciles without picking, and that worktree stays on today's directory against D3.
    fix: In card 5, state that `CreateWorktree`'s complete branch also calls `pickAccount` with the slug when the account file holds no entry for it, and add a row to `TestWire_CreateWorktreePicksTheNewWorktreesAccount` where a complete worktree with no entry is picked and one with an entry is not.
  - id: F13
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/07-known-config-dirs.md:71
    summary: Card 7 has reedcli's enterSession resolve the hub's prime location for SessionNamers' dirs, falling back to empty dirs, though enterSession already holds the watched worktree's location, whose dirs carry that worktree's env module value.
    fix: In card 7's caller list, replace the enterSession clause with `reedcli`'s `enterSession` passing `fabricengine.ConfigDirs(location)` of the watched session's location it already resolves, and drop the `configengine.Dirs{}` fallback.
---

# Plan review, round 1

The plan carries every decision of the record in a card, and the cards run in an order the code allows.
Three blocking defects remain: a launch site card 3 misses, usage limits on the skill-load turn that no card handles, and a registry test card 2 breaks without listing it.

Focus verdicts in brief:
cards 11 to 15 carry D7's steps 1 to 4, the bound on attach and the single-lock stop forms, and the added-after-Discussion ruling's predicate, `StopStepRuns` override and events-proof-off wait sit in cards 12 and 14.
Card 14's deferred-prompt rule goes past D7 (F7).
No card uses a symbol a later card introduces.
No card edits the board entry or files a task, and no card has a `**Verify:**` line.
The Edit glyphs on cards 3, 6 and 7 are member glyphs, and the re-signed members carry their arrows.

### [BLOCKING] Card 3 misses the dobby tidy agent's launch wiring

**Location:** `_lym/plan/03-engine-worktree-geometry.md:40`

**Issue:** `primeShuttleRunner` (`internal/battencli/cancelcmd.go:517`) builds engines with `shuttleengines.New`, and `battencli.ShuttleRunner` (`dobbywire.go:40`) returns it to `beamcli`'s `runnerTidyShuttle`, whose `Start` calls `StartGated` in the beam worktree (`internal/beamcli/dobbydeps.go:833-839`).
Card 3 switches only eight launching members, so this launch resolves only the hub pin.
The handler found the same gap on its own.

### [BLOCKING] Usage limits on the skill-load turn bypass the switch and the limit record

**Location:** `_lym/plan/14-in-run-relaunch.md:39`

**Issue:** `launch` calls `loadSkillsThenPrompt` before `Wait` (`internal/shuttleengine/run.go:784`).
`readLoadTurn` counts every turn end `TurnEndActivity` reads as an API error (`internal/shuttleengine/loadturn.go:220-240`), and claude's `newestTurnEndError` reads a usage limit as one (`internal/shuttleengine/claudeengine/activity.go:169-189`).
The load hold retypes up to its cap, and `abandonLoad` then ends the run `OutcomeAPIError` (`loadturn.go:290-318`).
On an exhausted account, a new session's first turn is the load turn, so this is the common way a run meets a limit.
D5's `blocked_until` and D7's switch never run there.

### [BLOCKING] Card 2 breaks the config registry's machine-key test

**Location:** `_lym/plan/02-claude-account-config.md:8`

**Issue:** `TestModules_EveryTemplateKeyClassified` (`internal/configreg/configreg_test.go:198-215`) wants an empty `MachineKeys` for every module it does not name, claude included.
The test is outside card 2's packages, so the card gate passes, and the batch and webster gates then fail.

### [MEDIUM] Card 16's start-path refused resume sits at a site that never sees the first API error

**Location:** `_lym/plan/16-refused-resume-starts-fresh.md:27`

**Issue:** `loadSkillTurn` loops on `readLoadTurn`, which records each API-error turn end in the load hold and retypes (`internal/shuttleengine/loadturn.go:136-160`).
`loadSkillsThenPrompt` gets control back only on a passed load or the cap error.

### [MEDIUM] Card 5's adopt seam panics in the existing adopt tests

**Location:** `_lym/plan/05-account-pick-at-making.md:24`

**Issue:** `world.deps()` (`internal/adoptcli/adopt_test.go:124`) builds the `adoptDeps` every adopt test uses, and it will not set `PickAccount`.
`runForeground` and `takeOverRuns` then call a nil func after each `AddPair` (`internal/adoptcli/foreground.go:124,326`).

### [MEDIUM] Doc files a card rewrites are listed under Extends

**Location:** `_lym/plan/12-run-relaunching-record.md:14`

**Issue:** The rubric flags `Extends` on a ref the card rewrites.
Sentences each card falsifies: `claudeengine/doc.go:142`, the plugins file and config directory sentences there, and its single-registry resume text (cards 7, 8); `shuttleengines/doc.go:30-31` (card 7); `reedengine/doc.go:466-471` (card 11); and `shuttleengine/doc.go`'s stop verb and carry-over text (card 12).
The background's done-when paragraph says resume drops or relaunches every non-live strand, which card 11's held strand breaks.

### [MEDIUM] Card 14's deferred-prompt rule cannot work as written and departs from D7

**Location:** `_lym/plan/14-in-run-relaunch.md:48`

**Issue:** `Spec.SkipSkillLoadTurn` keeps the prompt pointer on the launch line (`internal/shuttleengine/spec.go:156`), so step 2's continue line is the first turn whatever the record holds.
D7 step 2 and its criterion say the relaunch's first turn is the continue line and its `PromptLine` is empty.
The rule comes from the plan's own lens review, not from the record.

### [MEDIUM] Card 11's hold has no stated bound

**Location:** `_lym/plan/11-reed-hold-and-relaunch-ops.md:38`

**Issue:** The hold is a new exemption from reed's resume, its done-when drop and reconcile.
The card says what it skips but not what ends a hold whose holder died; that bound lives only in cards 12 and 15.

### [LOW] Card 4's resolution test repeats card 6's public-surface rows

**Location:** `_lym/plan/04-claude-account-store.md:21`

**Issue:** Five of `TestResolveAccount`'s six rows are rows of `TestPrepare_ExportsTheResolvedAccount` (card 6), which reaches `resolveAccount` through `Prepare`.
The two remaining rows move to card 6's test.

### [LOW] `lym shuttle state` does not report a relaunching run

**Location:** `_lym/plan/12-run-relaunching-record.md:23`

**Issue:** `stateCmd` maps each reading through `stateFields(reading.State)` plus strand, guid and history (`internal/shuttlecli/state.go:45-62`), so the new flag never reaches its output.

### [LOW] A held strand becomes removable at sites card 12 does not name

**Location:** `_lym/plan/12-run-relaunching-record.md:27`

**Issue:** `removeSupersededStrands` (`internal/shuttleengine/attach.go:233-245`) and the untied branch of `StopStepRuns` (`stepstop.go:98-101`) both filter on `strandRemovable`.

### [LOW] A re-entered Worktree-Create row never picks

**Location:** `_lym/plan/05-account-pick-at-making.md:44`

**Issue:** The `complete` branch of `CreateWorktree` returns `reconcileTaskPair` before `top.Add` (`internal/battencli/wire.go:1136-1145`), so a crash after the add skips the pick for good.

### [LOW] Card 7 resolves the wrong location for the watchdog's namers

**Location:** `_lym/plan/07-known-config-dirs.md:71`

**Issue:** `enterSession` already holds `location` from `resolveWatchedSession` (`internal/reedcli/watchdog.go:332-337`).
The prime's dirs miss a per-worktree `env` value, and the empty-dirs fallback silently reads only `~/.claude`.

## Rejected

- guard-reach finding 4, the `sweepOrphans` keep guard is unreachable: a real input reaches it. `lym reed remove`, reed's own remove verb, drops the strand from `reed.json` without touching the run record, so a relaunching record within its bound can face a sweep whose GUID set lacks its strand (`internal/shuttleengine/run.go:1186-1206`). D7 also requires the guard.
