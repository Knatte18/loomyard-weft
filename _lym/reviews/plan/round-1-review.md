---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/10-kwinplace-place-flow.md:40
    summary: "Mode `off` runs `RecordReport` from the window-wait snapshot before `kwinplace.lock` is taken, so a stale snapshot taken before another child's locked apply makes `updatePlaces` read that apply's moved windows as hand moves and rewrite their homes; D10 puts the record step under the lock."
    fix: "In card 10, move mode `off` after step 4: the `off` child takes `kwinplace.lock` through `acquireWithin` from the shared 120-second budget, runs a fresh `Query` under the lock passing the lock time as `after`, calls `RecordReport` from that snapshot with nothing moved, releases the lock and runs `afterPlace`. It skips the parent wait, since it plans nothing. Change the `TestFlow_Place` row `mode off records the place and calls no Apply` so it also asserts that the record follows the locked query, and add a row in which a concurrent flow's apply lands between the off child's window-wait query and its record without its placed window's home being rewritten."
  - id: F2
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/09-kwinplace-planner.md:33
    summary: "D8 says the prime's own window starts at the current desktop under either mode, but card 10 only sets the prime's anchor to the current desktop and card 9 starts `next` at the anchor plus one, with no input that tells the prime apart, so the prime's window under `next` lands one desktop past the current one."
    fix: "In card 9, add a `Prime` field to `PlanInput` and change the start-desktop bullet: a homeless prime window starts at the anchor (the current desktop) under both `same` and `next`. Add a `TestPlan` row in which the prime under `next` with room on the current desktop lands on the current desktop. In card 10, say that `Place` passes `req.Prime` into `PlanInput`, and extend the `TestFlow_Place` row `the prime's own window waits for no parent and anchors on the current desktop` so that under `next` it lands there too."
  - id: F3
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/10-kwinplace-place-flow.md:51
    summary: "No card builds the `ReportUpdate` that `RecordReport` takes. `ApplyReport` carries no pid, class or caption for `Window.Matches`, and zones only for the target desktop. A clamped window, released from its tile on every 1-row panel desktop, has no leaf listing it, and card 9 keeps D6's frame-corner zone rule private to `Plan`. So D9's current order and renumber after an apply have no carrying function."
    fix: "In card 10, add a function, such as `plan:internal/kwinplace#reportUpdateFrom`, that builds the `ReportUpdate` from the locked `Snapshot`, the `ApplyReport`, the places and the request. It joins each reported window to the snapshot's window by internal id for pid, class and caption, matches it to a worktree through `Window.Matches` against the places entries, and takes its desktop from the apply report. It takes each desktop's current order by D6's rule: the leaf listing the window, else the zone its frame's top-left corner starts at inset by padding, with untiled windows last in stacking order. The target desktop's zone boxes come from the apply report and every other desktop's from the snapshot. In card 9, export that current-order rule as one function, such as `plan:internal/kwinplace#CurrentOrder`, which `Plan` and the new function both call, with its rows moved from `TestPlan`'s ordering rows into its own table where they pin it. Under mode `off` the same function builds the update from the snapshot alone. Add `TestFlow_Place` rows asserting that after an apply on a 1-row grid under a panel, a reopen in key order leaves every order key unchanged."
  - id: F4
    severity: BLOCKING
    class: decision
    origin: handler
    location: _lym/plan/05-kwinplace-scripts-readers.md:93
    summary: "The plan leaves the KWin facts it rests on to be found while it runs. The Shared Decision probes-before-code puts the Open-risks probes before card 1 with a rule to halt on a contradicted fact, but no card carries that step, so nothing in the plan reaches it. Card 5 and card 7 take recordings mid-card, the moving ones only after the operator approves through the parent, and their tests cannot run without them."
    fix: "Parent decision, through `lym loom decision add`: have the Open-risks probes run now, before the plan is approved, by a session that reaches the user session bus for the read-only ones and by or with the operator for the moving ones. Record each fact's outcome in the decision record, and reopen any decision a contradicted fact rests on. In the same sessions, take every recording that card 5 (neve-kscreen, neve-kscreen-laptop-off, neve-query, neve-query-no-output, neve-apply, neve-apply-one-row) and card 7 (neve-qdbus) name, with their source.yaml lines. Then replace the probes-before-code Shared Decision with the recorded outcomes, which may need a fresh Plan-Write since the overview outside its Card Index is frozen. Cards 5 and 7 then commit the recordings already taken and carry no operator step."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:affected-set
    location: _lym/plan/11-place-window-child.md:41
    summary: "Card 11 registers every non-prime placement child under the worktree's slug, which goes beyond D10's registration only when the opening verb runs inside a run, since `lym machine start` and a hand-run `lym reed window` run in none. It also weighs only `End`, yet `requireStillOrphan`, the orphan view's `ProcessLive` and cancel's `otherLeft` count any live record under the slug for up to the child's 10 + 180 + 120 seconds plus its script bounds."
    fix: "In card 11, list each reader of a slug's live records (`internal/pairteardown/leftover.go` `requireStillOrphan`, `internal/battencli/leftoverpairs.go`'s `ProcessLive`, `internal/battencli/cancel.go`'s `otherLeft`, `internal/pairteardown/teardown.go`'s `ownedProcesses`) and state for each why counting a `RolePlacement` record is right, or give each reader that must pass over it a role check and a row in its existing test. Either limit registration to an opening verb that runs inside a run, or have the parent record through `lym loom decision add` that every non-prime child registers, which the Shared Decision run-registration then cites."
  - id: F6
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/07-kwinplace-script-runner.md:40
    summary: "D3 says every child process is logged, waited on and bounded by a timeout, but card 7's exec seam bounds none of its `qdbus6`, `journalctl` or `kscreen-doctor` calls. A hung `loadScript` or `run` then holds the per-user script lock until SIGTERM, and every placement on the machine fails at its 30-second acquire."
    fix: "In card 7, give the `Runner` a per-call timeout field, with a production value such as 5 seconds and a short one in tests. The exec seam runs each call under a context bounded by it, a timed-out call fails the run, and the run still unloads and releases as step 7 says. Add a `TestRunner_ScriptRuns` row in which the exec answers a load call only after its bound, asserting that the run fails, unloads by plugin name and releases the lock."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/11-place-window-child.md:83
    summary: "The `UnknownWorktree` row cannot reach the resolution-failure branch it claims. `lymcwd.NamedWorktree` answers anchor `.` for an unrecorded name, `TaskSettingChain` skips a missing root and `ParentOf` answers the prime, so no resolution fails, and the row passes only because the window command starts in a missing directory. The branch's real inputs, a refused `beam.window_placement` or desktop key, reach no row."
    fix: "In card 11, replace the `UnknownWorktree` row with two rows of `TestPlaceWindowChild`. One sets the task's `beam.window_placement` to a refused value through the worktree file; the other sets a refused `desktop.window_placement` through `configengine.Set` on the machine layer. Each asserts that the task's places entry is settled and that the child's log warning names the way forward the card prescribes. Either drop the unknown-worktree case, or have the verb check the told name against `fabricengine.List` as a resolution step and keep the row for that check."
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/10-kwinplace-place-flow.md:53
    summary: "D12 runs `window_command` on every exit but SIGTERM once the child is handed a window. Card 10 runs `afterPlace` only on exits of steps 2 to 6, so a failed step-1 `RecordOpen` exits with no settle and no window command. Card 11 does not say what a failed `NewRunner` (`UserStateDir`) in the verb does."
    fix: "In card 10, change step 7 to every exit of steps 1 to 6, with a failed `RecordOpen` logged as a warning and its later `Settle` attempted, and add a `TestFlow_Place` row whose places path is a file under a read-only directory made through `permkit.Chmod`, asserting that `afterPlace` ran once. In card 11, add `NewRunner` failing to the resolution failures that settle the entry and run `afterPlace` where the config dirs resolved."
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/07-kwinplace-script-runner.md:26
    summary: "Card 7 lists `pattern/PATTERN-test-speed.md` under Extends but rewrites its tier table's `tier3` row. It also leaves `docs/benchmarks/running-tests.md`, which defines `tier3` as tests that spawn a real `claude` session (lines 34 and 55), untargeted and stale."
    fix: "In card 7, move `//pattern/PATTERN-test-speed.md` from `**Extends:**` to a new `**Edit:**` section and add `//docs/benchmarks/running-tests.md` there, with its Intent saying both lines name a real desktop session beside the LLM session. Checked with `lym loom validate-plan --overlay`, which reports no new blocking finding."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/07-kwinplace-script-runner.md:103
    summary: "The tier3 contract test and the tier1 tests of cards 11 and 12 name no test file, though they sit beside tier0 tests of their packages, and PATTERN-test-speed's one-tier-per-file rule, held by `cmd/lym/tiertags_test.go`, fails a test placed in one. Card 12's tier1 test would naturally go into its tier0 `window_test.go`."
    fix: "In card 7, name a tier3 file for `TestContract_TheRealKWinAnswersAsRecorded`, such as `internal/kwinplace/contract_test.go`. In card 11, name a tier1 file for `TestPlaceWindowChild`, such as `internal/reedcli/placewindowchild_integration_test.go`. In card 12, name `internal/reedcli/placewindow_integration_test.go`, a tier1 file, for `TestHandTerminalWindow_DirectoryInNoWorktreeHandsNothing`."
  - id: F11
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/10-kwinplace-place-flow.md:45
    summary: "Card 10 leaves two inputs of its locked steps unstated. It names no `within` for its `Query` calls, though D3 bounds a window-wait query's script-lock take by what remains of the window wait. It does not say that the places `Anchor` and `Plan` read are re-read under the lock, which D8 requires."
    fix: "In card 10, say that window-wait queries pass the window wait's remaining time as `within` and the locked query passes the full 30 seconds, and that step 5 reads the places file with `ReadPlaces` after the locked `Query` and hands that read to `Anchor` and `PlanInput`. Add the `within` value to the existing window-wait row's assertions."
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/12-reed-window-placement.md:8
    summary: "Card 12 moves the window command into the placement child under placement, but `internal/reedcli/window.go`'s file comment still says the verb then runs the machine's window command, and no card targets it."
    fix: "In card 12, replace the Edit targets `internal/reedcli#openTerminalWindow` and `internal/reedcli#reedCLI.windowCmd` with `internal/reedcli/window.go#`, since the card changes both functions and the file comment. Its Intent then says the file comment names the hand-off to the placement child under placement. Listing the file beside the members is refused by `redundant-file-target`; this replacement was checked with `lym loom validate-plan --overlay` together with F9's fix, and it reports no new blocking finding."
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: docs/overview.md:346
    summary: "The package tree in `docs/overview.md` gains no `internal/kwinplace/` line, and its `internal/desktophook/` line still names only the window command run after `lym reed window` and `lym ide`. Cards 12 and 13 target `docs/overview.md` but edit only its window paragraphs."
    fix: "In card 12, which already targets `//docs/overview.md`, add to its Intent a package-tree line for `internal/kwinplace/` naming the KWin and kscreen boundary and the placement flow, and widen the `internal/desktophook/` line to name the placement keys and the cancellable window command."
  - id: F14
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-kwinplace-places.md:37
    summary: "`RecordOpen` keeps `SettledBy`, and `Settled` is `SettledBy == PID`, so a new window whose pid equals the entry's previous one, as after a reboot, reads as settled at once and its children anchor on its stale entry."
    fix: "In card 8, have `RecordOpen` set `SettledBy` to 0 as it sets the new pid, and add a `TestPlacesFile` row in which `RecordOpen` with the entry's own previous pid leaves the entry unsettled."
  - id: F15
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-kwinplace-places.md:42
    summary: "The `ErrDecode` heal removes the file and retries outside `state.UpdateJSON`'s lock, so two writers that both meet an undecodable file can each remove it, and the second removal drops the first writer's freshly written entry."
    fix: "In card 8, run the heal under the places lock: take `places.lock` with `lock.AcquireWriteLock`, re-read the file and remove it only if it still fails to decode, release the lock, then rerun the update. Extend the `TestPlacesFile` undecodable row to two concurrent writers, asserting that both entries survive."
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/10-kwinplace-place-flow.md:45
    summary: "Step 5's branch for a homed window whose home the locked snapshot no longer holds, which then needs an anchor and returns to the parent wait, is reachable when the operator removes a desktop between the wait and the lock. No `TestFlow_Place` row reaches it."
    fix: "In card 10, add a `TestFlow_Place` row in which the window-wait snapshot holds the window's home and the locked `Query` does not, asserting that the child waits for its parent's settled entry and anchors on the parent's desktop."
  - id: F17
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/11-place-window-child.md:67
    summary: "Card 11's attack-surface bound lists only what the verb reads and moves. Any local process that runs the hidden verb can also make it write the places file under any told `--hub` path, write `kwinquery.json` and `kwinscript.loaded`, and run the told worktree's `desktop.window_command`."
    fix: "In card 11, extend the bound: the verb writes only the told hub's places file, which `RecordOpen` creates under `<hub>/_bolt/.lym/window-places/`, and the files beside the per-user script lock. It runs only the `desktop.window_command` the told worktree's own config stack names, which the same user could run directly."
  - id: F18
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/07-kwinplace-script-runner.md:103
    summary: "PATTERN-test-stand-ins holds every recording to a contract test that compares the real thing with it. The tier3 contract test runs only `Query` and `ReadTilingOutput`, so the apply recordings and the `neve-qdbus` load-taken and loaded-true answers have no contract comparison and no way to be recorded again."
    fix: "In card 7, extend `TestContract_TheRealKWinAnswersAsRecorded` with steps that are read-only or self-reverting: a second load of a still-loaded plugin name compared with `load-taken.txt`, `isScriptLoaded` before and after unload compared with `loaded-true.txt` and `loaded-false.txt`, and an `Apply` on a throwaway window and a desktop that is not current, compared by kind and fields with `neve-apply`. Mark the apply step as one the operator runs or approves."
---

The plan carries most of the record, but its child flow has four structural gaps, F1 to F4.
F1, F2 and F3 are design errors in the flow and planner cards.
F4 is a call for the parent: the facts the plan rests on, and the recordings its tests need, are left to be found during implementation.

### [BLOCKING] Mode off records from a stale snapshot outside the lock

**Location:** `_lym/plan/10-kwinplace-place-flow.md:40`

**Issue:** Step 3 runs `RecordReport` for mode `off` from the window-wait snapshot, which may have been shared by another child, before step 4 takes `kwinplace.lock`.
`updatePlaces` treats a settled window whose desktop differs from its `LastDesktop` as moved by hand.
Suppose another child's locked apply lands between the `off` child's query and its write.
The stale snapshot then shows that child's placed window on the desktop KWin opened it on, and the rule rewrites its home there.
D10 puts the query, plan, apply and record steps under the lock, and D9 lists the record-only run beside the post-apply write.
Both the fork and my own reading reached this independently.

### [BLOCKING] The prime's window under next starts one desktop past the current one

**Location:** `_lym/plan/09-kwinplace-planner.md:33`

**Issue:** D8 says the prime's own window, having no parent, starts at the current desktop under either mode.
Card 10 gives the prime the current desktop as its anchor.
Card 9 then starts `same` at the anchor and `next` at the anchor plus one, and `PlanInput` carries no field that tells the prime apart.
So the prime's window under `next` starts one desktop past the current one.
No row of either card pins the prime's start under `next`.

### [BLOCKING] No card builds the report update the places rules run on

**Location:** `_lym/plan/10-kwinplace-place-flow.md:51`

**Issue:** Card 8's `ReportUpdate` asks for each reported window of the hub, matched through `Window.Matches`, with its desktop and current zone order.
Card 10 only says "`Apply`, then `RecordReport`".
`ApplyReport` (card 5) holds internal ids, desktops, the zone on the target desktop, a tiled flag and the frame, but no pid, resource class or caption, so it cannot feed `Window.Matches` alone.
It holds zones only for the target desktop, while D9's hand-move renumber needs the current order of other desktops.
After the apply's clamp, a released bottom-row window has no leaf that lists it.
On Neve's bottom panel that is every window of a 1-row desktop.
D6's rule, the zone the frame starts at, lives only inside card 9's `Plan`.
D9's post-apply renumber and its "a reopen that keeps the keys' order keeps the keys" criterion therefore rest on a function no card defines.

### [BLOCKING] The plan defers its research steps and operator recordings into implementation

**Location:** `_lym/plan/05-kwinplace-scripts-readers.md:93`

**Issue:** The decision record lists eight unobserved KWin facts.
D3's apply, D6's grid and clamp, D4's output match, D1's probe, D3's end-of-run guarantee and script lock, and D2's match rest on them.
The overview's Shared Decision probes-before-code runs them "before card 1" with a rule to halt when one is contradicted.
That is the research step and halt rule the rubric bars, and no card carries the step, so nothing the plan executor runs reaches it.
Card 5 also takes its recordings while it runs: `neve-apply`, `neve-apply-one-row` and `neve-kscreen-laptop-off` only once the operator approves through the parent.
Its own tier0 test, and card 7's, cannot pass without them.
The record makes the probes the implementing run's first step.
That is why this is a `decision` for the parent, not a fixer edit: the facts and recordings belong in the record before approval.

### [MEDIUM] Placement-child registration beyond D10, and readers of the slug's records

**Location:** `_lym/plan/11-place-window-child.md:41`

**Issue:** D10 registers the child "when the opening verb runs inside a run".
The Shared Decision run-registration, carried by card 11, registers every non-prime child under the worktree's name, including those of `lym machine start` and a hand-run `lym reed window`.
The card weighs only `End`.
Other readers count any live record under a slug: `pairteardown/leftover.go:172-178`, `battencli/leftoverpairs.go:66-70` and `battencli/cancel.go:782-787`.
They will now see a placement child as a live run process for minutes.
This merges the affected-set lens's F2 with my own scope check of the Shared Decision.

### [MEDIUM] The script runner's child processes carry no timeout

**Location:** `_lym/plan/07-kwinplace-script-runner.md:40`

**Issue:** D3 ends with "Every child process is logged, waited on and bounded by a timeout".
Card 7's exec seam logs each spawn, but only the waits around its calls are bounded: the 5-second journal loop and the 2-second unload wait.
A `qdbus6 ... run` or `loadScript` call that hangs holds `kwinscript.lock` with no bound.
The child's context is cancelled only by SIGTERM.

### [MEDIUM] The UnknownWorktree row does not reach the branch it tests

**Location:** `_lym/plan/11-place-window-child.md:83`

**Issue:** I confirmed that `anchorRelOrRoot` answers `.` for an unrecorded name (`internal/lymcwd/lymcwd.go:185-193`), so `NamedWorktree` does not fail.
The child runs its normal flow, fails at `kscreen-doctor` and runs the window command in a directory that does not exist.
The row's "no marker" then holds for a reason the card does not name.
The verb's prescribed warnings for a refused `beam.window_placement` or desktop key have no reaching test.
The guard-reach fork rated this `BLOCKING`.
It is a coverage defect over a branch whose behaviour the card does state, so it is held at `MEDIUM`.

### [MEDIUM] The window command is skipped on two child exits

**Location:** `_lym/plan/10-kwinplace-place-flow.md:53`

**Issue:** D12 says that once handed a window, the child runs `window_command` on every exit path but SIGTERM.
Card 10 bounds `afterPlace` to exits of steps 2 to 6, leaving a failed step-1 `RecordOpen` (an unwritable places directory) with neither a settle nor the window command.
Card 11 does not say whether a `NewRunner` failure (`standalonestate.UserStateDir` erroring) counts among the resolution failures that settle and run `afterPlace`.

### [MEDIUM] Extends on a pattern table row the card rewrites, and a stale tier doc

**Location:** `_lym/plan/07-kwinplace-script-runner.md:26`

**Issue:** The card changes the `tier3` row of PATTERN-test-speed's tier table, an existing cell, so the file is an `Edit`, not an `Extends`.
`docs/benchmarks/running-tests.md:34` and `:55` define `tier3` as real `claude` and LLM sessions, and no card targets them.
This merges the repo-invariants lens's F3 with the Extends check.

### [LOW] Tests whose tier differs from their package's other tests name no file

**Location:** `_lym/plan/07-kwinplace-script-runner.md:103`

**Issue:** `cmd/lym/tiertags_test.go` fails a test file whose functions name two tiers.
The cards do not keep the tier3 contract test or the tier1 tests of cards 11 and 12 apart from the tier0 tests of their packages.
`internal/reedcli/window_test.go` is tier0.
This merges the repo-invariants lens's F1 and F2, with F1 lowered from `MEDIUM`: the scan catches a mistake, and the fix is only the file name.

### [LOW] Card 10 leaves the within values and the locked places read unstated

**Location:** `_lym/plan/10-kwinplace-place-flow.md:45`

**Issue:** Card 7's `Query` takes `within`, and D3 bounds a window-wait query's script-lock take by what remains of the window wait.
Card 10 names no `within` value for its queries.
D8 reads the anchor under the lock, and card 10 never says where the places that `Anchor` and `PlanInput` take are read.

### [LOW] reedcli's window.go file comment goes stale

**Location:** `_lym/plan/12-reed-window-placement.md:8`

**Issue:** Lines 1-5 of the file comment say the verb runs the machine's window command through `internal/desktophook`.
Under placement the child runs it.
The gate refuses adding the file beside its members, so the proposed fix lists the file in place of the two members.

### [LOW] The overview's package tree misses kwinplace

**Location:** `docs/overview.md:346`

**Issue:** The tree lists each package with its concern.
The new package gets no line, and desktophook's line no longer covers what the package holds.

### [LOW] RecordOpen keeps SettledBy, so pid reuse reads as settled

**Location:** `_lym/plan/08-kwinplace-places.md:37`

**Issue:** D9 has the pre-wait write leave the entry "unsettled until its child finishes".
With `SettledBy` kept, a new window whose pid equals the entry's previous pid reads as settled at once.
D2's rationale names exactly that pid reuse after a reboot.

### [LOW] The undecodable-file heal races between writers

**Location:** `_lym/plan/08-kwinplace-places.md:42`

**Issue:** `state.UpdateJSON` releases its lock when it returns `ErrDecode`.
The card's remove-and-retry then runs outside the lock, so the D9 guarantee that interleaved writes keep each other's entries fails on the heal path.

### [LOW] No flow row reaches the vanished-home branch

**Location:** `_lym/plan/10-kwinplace-place-flow.md:45`

**Issue:** Step 5's "A homed window whose home the fresh snapshot no longer holds needs an anchor from then on" is a branch of the flow with no row.
Card 9's `TestPlan` row covers only the planner's fallback to the anchor.

### [LOW] The hidden verb's bound leaves out what it writes and runs

**Location:** `_lym/plan/11-place-window-child.md:67`

**Issue:** The card states what the verb lets through, but only as reads and moves.
It also creates and writes files under a told hub path and the per-user state directory, and it runs a worktree's configured window command.
The bound should name both.

### [LOW] The contract test covers only part of the recordings

**Location:** `_lym/plan/07-kwinplace-script-runner.md:103`

**Issue:** PATTERN-test-stand-ins requires a contract test comparing the real thing with its recording.
The apply journals and the `neve-qdbus` answers other than a load's id are never compared or recorded again, so a KWin change to them goes unseen.

## Ground truths and settled focus questions

- PATTERN-test-speed puts a behaviour at the lowest tier its real modules run in, so the Shared Decision tier0-for-pure-rules is right against the record's "tier-1 tests".
  This is a constraint conflicting with the record, and the constraint wins.
- Card 10's `after` values (the wait's start, then the previous run's end, then the lock time under the lock) are stricter than D10's "after its previous query, or its wait, began".
  Read literally, the record's wording would have a child reread its own last report, so the card's choice is no defect.
- Card 11's reduced `PATH` keeps the child off a real desktop: the flow reads the tiling output through `kscreen-doctor` before any `qdbus6` call, and that lookup fails first.
- SIGTERM paths: the runner unloads under a context of its own and releases.
  `acquireWithin`'s abandoned acquire dies with the process, so a flock held by it is released by the OS.
  `AfterPlacement`'s group kill fits inside `runproc.End`'s 5-second SIGKILL.
- The tier0 estimates sum to about 4.2 seconds, under the 60-second plan budget.
- Acceptance criteria: every bullet maps to a planned row once F1, F2, F7 and F8's rows are added.
  The two Review bullets map to cards 1, 2 and 12.

## Rejected

- affected-set F3, the stencil-seed annotation on `place-window`: `window`, the other launcher-run reed verb, carries no such annotation either, and no harm from the seed pass in the hub directory was shown.
- affected-set F4, the hub `gate.yaml` tier description: it lives in the Bolt checkout outside the task's worktree, so no card can carry it and it is no plan defect.
- Member glyphs on card 11's `internal/reedcli/cli.go#`: the card changes `cli.go`'s file comment, which only the self glyph covers.
  `lym loom validate-plan --overlay` refuses `internal/reedcli#Command` beside it under `redundant-file-target`, so the self glyph is the right target.
- guard-reach's card 4 note on `Open` with `separate` and no Konsole: a fail-loud wiring guard in `refusal-spec.md`'s sense, taking no reaching test.
