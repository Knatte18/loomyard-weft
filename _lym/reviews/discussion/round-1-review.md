---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:95
    summary: "D5's way forward for the `lym loom start` refusal, a board upsert to tapestry, cannot clear it: the start reads its recipe from the worktree's seed, which stays jacquard"
    fix: "In D5, give each judging site its own way forward. Batten's start guard keeps `lym board upsert '{\"slug\":\"<slug>\",\"recipe\":\"tapestry\"}'`. The `lym loom start` refusal names D4's jacquard sequence: from the hub's prime worktree `lym batten cancel <slug> --remove`, `lym board upsert '{\"slug\":\"<slug>\",\"recipe\":\"tapestry\"}'`, `lym board set-status '{\"slug\":\"<slug>\",\"status\":null}'`, then `lym batten run <slug>`. Drop the clause saying a start with no status file can take the upsert, or confine it to batten's site. Change the D5 acceptance criterion from 'both messages carry the way forward' to: each site's message carries its own way forward, and the test reaches both."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:82
    summary: "D4 sends a jacquard run down the RecordWrite path of the decision verbs, whose refusals name `lym loom goto --to Record-Write` and whose pending test reads Record-Write by name; jacquard has no such row"
    fix: "In D4, after 'The decision verbs treat RecordSeed as they treat RecordWrite', add this. The RecordWrite-path refusals in `internal/loomcli/decision.go` (`decisionWayRecordStart`, `decisionWayRecordResume`, `decisionWayRecordGoto`, `decisionWayUncommittedGoto`) and `recordWritePending` name the row of the run's recipe that runs the record or seed engine, read from the recipe's rows through `loomrecipe.EngineRow`, never the literal Record-Write. Add an acceptance criterion: on a jacquard run past Record-Seed with an uncommitted record, `lym loom decision add` refuses with a way forward naming `lym loom goto --to Record-Seed` (tier 1)."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:98
    summary: "D5 says an entry with a stored record starts, but a jacquard start of an entry its own motif run discussed finds that run's done status and is refused 'has already completed'. Batten clears a done motif run only for a raddle start, so D5's no-status-file guard is never reached"
    fix: "In D5, settle the motif-then-jacquard start on one slug. Recommended: batten's done-motif-run clear in `internal/battencli/startguard.go` (`startClearsMotifRun`, judged today by `recipe != shedrun.RecipeRaddle`) also admits a start whose recipe needs a stored record, judged by D5's predicate over the rows. That start is then judged as fresh, and D5's refusal applies to it. Add a tier 1 criterion: after a done motif run, `lym batten run` of the same entry set to jacquard starts. The alternative is to state that such a start meets `doneSlugRefusal` and that the operator runs `lym batten cleanup --apply` first."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:122
    summary: "D7's reconcile claim leaves two gaps. A hub that removed the `solid` profile still gains `defaults.jacquard: solid`, which runprofile.Load refuses, so profiles.yaml stops loading. A profile the operator added gains no jacquard block, so raddle refuses jacquard parts that name it"
    fix: "In D7, add the two cases and their handling. (a) A gained `defaults.jacquard` that names a profile with no jacquard block is dropped by a one-time reinterpretation, as the arras default's was. (b) An operator-added profile that holds a tapestry block either gains a jacquard block, made from that tapestry block minus `loom.discussion`, through the same reinterpretation, or D6's bound names the way forward `lym config profiles --set <profile>.jacquard=<block>`. Add the case to the D7 acceptance criterion (tier 0, `internal/runprofile`). Unverified: how `internal/runprofile/reinterpret.go` registers a reinterpretation; only `doc.go`'s account of the arras one was read."
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:41
    summary: "D2 says tapestry's top-level keys equal jacquard's, but D2 sets tapestry's entry to Motif and D1 sets jacquard's to Preflight, so the D1/D2 criterion asserting equal keys fails by construction"
    fix: "In D2, replace 'It keeps its own top-level keys, since an expansion reads none of a block's, and they equal jacquard's.' with 'It keeps its own top-level keys, since an expansion reads none of a block's; all but `entry` equal jacquard's.' In the D1/D2 acceptance criterion, replace 'jacquard's top-level keys equal tapestry's' with 'jacquard's top-level keys other than `entry` equal tapestry's'."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:56
    summary: "D3 admits any last outermost block row with an empty on_done. A gated one gets SkipTo empty, and skipTarget falls back to OnDone, the block's own entry, so a gate that does not hold enters the block instead of passing over it"
    fix: "In D3, admit an empty `on_done` on the last outermost block row only when its `block_gate` is `always` (`shedrecipe.BlockGateAlways`). Refuse any other gate in `shedbuild.Expand`, with the way forward 'set on_done, or use block_gate always'. State in the Bound that no skip can reach the empty exit. Add the refusal of a gated last block with no `on_done` to the D3 acceptance criterion (tier 0, `internal/shedbuild`)."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:99
    summary: "D5 places the `lym loom start` check 'before it writes the seed', but bootstrap rewrites the seed and runs loomshed.Seed on every start, re-entries included, so a re-entry of a jacquard run whose entry lost its record is refused, against 'A resume passes over it'"
    fix: "In D5's `lym loom start` bullet, state that the check runs only on a first start: the status file is absent when bootstrap reads it, before `loomshed.Seed` writes it. A re-run over an existing status file passes over the check, and Record-Seed's Stuck guards it. Add to the D5 acceptance criterion: re-running `lym loom start` on a seeded jacquard run whose entry has lost its record is not refused there (tier 1)."
  - id: F8
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md:22
    summary: "The board note `plan-and-build-recipes`, filed after this Discussion in the same project `plan-ahead-chains`, splits tapestry's post-Discussion rows into a plan recipe and a build recipe. That conflicts with D1's one jacquard recipe holding them all"
    fix: "For the parent: either keep D1 as the entry asks, with jacquard as one recipe of every row after Discussion and the note as later work that may split it, or redesign jacquard now to fit the note's plan/build split. Record the choice with `lym loom decision add`; the writer cannot settle it."
  - id: F9
    severity: LOW
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md:185
    summary: "The board entry asks the task to add the 'jacq' nickname; D11 and the Open risks line hand that write to the parent after landing, and no board note answers it"
    fix: "For the parent: accept D11, so the parent runs `lym heddle set` for the `nicknames` entry once the run lands, and record that with `lym loom decision add`. Or rule that a card of this run writes the nickname through `lym heddle set`, which changes D11 and drops the Open risks line."
  - id: F10
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:205
    summary: "The D11 Review criterion checks the change description, but tapestry's Code-Review perch runs before Describe writes it (Code-Bouncer to Drift-Recheck to Describe), so the code review cannot check it on the diff"
    fix: "Replace the bullet 'Review: D11 — the change description asks the parent to run `lym heddle set` for the nickname after landing.' with 'Review: D11 — no file of the diff writes Bolt's `heddle.json` or a nickname line.' The parent's own write stays under Open risks."
  - id: F11
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:52
    summary: "D2's rationale says Preflight's checks guard config only the build rows read, but Preflight also runs the worktree-clean, fabric-sync, fabric-ready and junction checks, and Discussion-Write commits through that fabric"
    fix: "Rewrite D2's second rationale sentence to: 'Preflight's batcher and drift checks guard config only the build rows read; its worktree and fabric checks also cover Discussion's commit, which motif already runs without, so a fabric fault in an undiscussed tapestry run now shows at Discussion first. A Preflight halt after Discussion keeps the approved record.'"
  - id: F12
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:141
    summary: "D9 keys under jacquard every tapestry run with stored_record true. stored_record is the entry's discussed flag at start, so a run flagged discussed whose record file was missing ran Discussion yet is counted under jacquard"
    fix: "In D9's Bound, add: 'A tapestry run of an entry flagged discussed whose record file was missing ran Discussion and is still counted under jacquard; that case is rare, and the cost cell accepts it.' Alternatively, have `runUsageRecord` in `internal/loomcli/sharedbootstrap.go` set `stored_record` only when the entry's record file exists, matching `ReadStored`."
---

### [BLOCKING] The `lym loom start` refusal names a way forward that cannot clear it

**Location:** `_lym/discussion/decision-record.md:95`

**Issue:** The batten guard and `lym loom start` share one way forward: `lym board upsert '{"slug":"<slug>","recipe":"tapestry"}'`.
`lym loom start` takes its recipe from the worktree's existing seed (`resolveSeedRecipe` returns `existing.Recipe`, `internal/loomcli/sharedbootstrap.go:105-109`, `:275-280`), and batten's Seed-Child writes that seed.
With no seed, the start resolves to tapestry, so the check fires only when a jacquard seed is present.
There, the board edit changes nothing, and the next start refuses again.
`PATTERN-refusal-way-forward` is broken at that site, and the record lists that pattern as a Constraint.
Four lenses found this.

### [BLOCKING] D4's decision-verb path names a row jacquard does not hold

**Location:** `_lym/discussion/decision-record.md:82`

**Issue:** D4 sends RecordSeed down the RecordWrite path of `lym loom decision add` and `replace`.
On that path, `checkDarnRecord` and `missingRecordRefusal` return ways forward fixed to the row name: "once Record-Write has run" and `lym loom goto --to Record-Write` (`internal/loomcli/decision.go:86-90`, `:626-629`, `:646-661`).
`recordWritePending` tests `loomshed.NameRecordWrite` by name (`decision.go:604-606`).
On jacquard that goto is refused as "names no producer in the list" (`internal/shedengine/goto.go`), so the refusals carry dead ways forward against `PATTERN-refusal-way-forward`.
D4 covers only which rows count as pending, not the texts.

### [MEDIUM] A jacquard start after the entry's own motif run is refused

**Location:** `_lym/discussion/decision-record.md:98`

**Issue:** The board entry names "a discussed entry" as a jacquard input, and the motif-then-jacquard flow is the one its Seen section describes.
The motif run leaves a done status under the slug.
`refuseAtStart` clears it only when the entry's recipe is raddle (`internal/battencli/startguard.go:178`), so `found` stays true.
D5's guard, which judges only a start with no status file, is never reached, and the start meets `doneSlugRefusal` (`internal/battencli/arm.go:945-949`).
D5 and its criterion "an entry with a stored record starts" are silent on this case.

### [MEDIUM] D7's reconcile leaves a broken default and uncovered profiles

**Location:** `_lym/discussion/decision-record.md:122`

**Issue:** `yamlengine.FollowTemplate` gains a key that both the file and its base lack, and it keeps a unit only the file holds (`internal/yamlengine/follow.go:45-52`).
On a hub that removed `solid`, the reconcile gains `defaults.jacquard: solid`.
`runprofile.Load` refuses a default that names a profile with no block for its recipe, so `profiles.yaml` stops loading, the case the arras default needed a reinterpretation for (`internal/runprofile/doc.go`).
A profile the operator added gains no jacquard block, so after D6 raddle refuses a jacquard part that names it, where its tapestry part passed.
This merges the reports of the existing-paths, interactions, walkthrough and bound-breaker lenses.

### [MEDIUM] D2's equal-keys claim contradicts the two entries

**Location:** `_lym/discussion/decision-record.md:41`

**Issue:** D1 sets jacquard's `entry` to Preflight, and D2 sets tapestry's to Motif, yet D2 and the D1/D2 criterion say the top-level keys are equal.
A test built from that criterion fails by construction.

### [MEDIUM] D3 lets a gated last block route its skip into the block

**Location:** `_lym/discussion/decision-record.md:56`

**Issue:** `Expand` emits the block row with `OnDone` set to the sub-recipe's entry and `SkipTo` set to the block's exit (`internal/shedbuild/expand.go:96-104`).
With an empty exit, `skipTarget` falls back to `OnDone` (`internal/shedengine/producer.go:130-134`), so a gate that does not hold enters the block.
Tapestry's Jacquard row uses `always` and never meets this, but D3 admits any gate, and its Bound does not state what the new edge lets through for other recipes.

### [MEDIUM] The `lym loom start` check's placement refuses a re-entry

**Location:** `_lym/discussion/decision-record.md:99`

**Issue:** Bootstrap writes the shedrun seed and calls `loomshed.Seed` on every `lym loom start`, and only `ErrSeedExists` tells a re-entry apart (`internal/loomcli/sharedbootstrap.go:105-127`).
D5 places the check "before it writes the seed".
That placement refuses a re-run of a started jacquard run whose entry has lost its record, which contradicts "A resume passes over it".

### [MEDIUM] Another board note splits the same rows differently

**Location:** `_lym/discussion/decision-record.md:22`

**Issue:** The note `plan-and-build-recipes` (filed 2026-11-28T12:10, project `plan-ahead-chains`) proposes that motif, a plan recipe and a build recipe make up tapestry.
D1 puts all of tapestry's post-Discussion rows in one jacquard recipe.
Whether jacquard is built as the note's future split or stays one recipe is a call only the parent can make.

### [LOW] The nickname write is handed to the parent

**Location:** `_lym/discussion/decision-record.md:185`

**Issue:** The entry asks the task to add "jacq" to the repository's nicknames.
D11 hands that write to the parent after landing.
The Open risks line raises it for the parent, and no `Decision for recipe-build-from-sketch:` note answers it.

### [LOW] The D11 Review criterion is unreachable at Code-Review

**Location:** `_lym/discussion/decision-record.md:205`

**Issue:** The discussion spec has the code review check a `Review:` criterion by reading the text the diff leaves.
Tapestry runs Describe after the Code-Review perch (`contracts/recipes/tapestry-recipe.yaml`: Code-Bouncer to Drift-Recheck to Describe), so the change description does not exist when that review runs.

### [LOW] D2's rationale overstates what Preflight leaves out

**Location:** `_lym/discussion/decision-record.md:52`

**Issue:** `preflightshed`'s `Call` runs the batcher and drift checks, which serve the build rows.
It also runs the worktree-clean, fabric-sync, fabric-ready and junction checks (`internal/preflightshed/preflight.go`), and Discussion-Write's commit goes through the fabric those checks cover.
The hub reconcile also runs on every config-loading verb (`internal/hubreconcile/doc.go`), and motif already runs Discussion with no Preflight.
So the decision stands, but its premise as written is false.

### [LOW] D9's bound misses a discussed entry with no record file

**Location:** `_lym/discussion/decision-record.md:141`

**Issue:** `runUsageRecord` sets `stored_record` from `entry.Discussed` alone (`internal/loomcli/sharedbootstrap.go:245`).
`seedDiscussedRecord` seeds only when the record file exists (`internal/loomcli/discussionrecord.go:64-67`).
A run flagged discussed with no file therefore runs Discussion and is still counted under jacquard.
