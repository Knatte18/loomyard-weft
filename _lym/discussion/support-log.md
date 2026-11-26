# Support log — fabric-git-free-core

## Interview

Autonomous mode: no operator answered in the pane; picks are auto-picks unless the parent relayed an operator decision, as the question ledger marks.

### Turn 1 — exploration

Four research subagents mapped the package, its tests and their cost, the gate scope and fabric's importers, and the motif and raddle recipes with the touching board entries.
Findings the design rests on:

- `internal/fabricengine` is one flat package of about 36k production and 58k test lines; nearly every verb interleaves read, decide and write against git.
  Plain-data pieces exist already: path classification, parent tips, verdict types, `Mutations`, and `ReconcileOrigin`'s injected seams.
- The package's tests make about 350 hub copies, 225 pairs with a push, 37 one-repository hubs and 16 second clones per run; tier1 call sites outnumber tier0 almost three to one.
  The speed docs show trims and sharding did not move the wall.
- The gate scope adds every package whose files or tests import a changed package; about 60 packages' tests and about 50 production packages import fabricengine, most only for path constants, options and result types.
- Raddle reads a Sketch's record and proposes parts; an order the record states in a decision binds it through `order_departures`; parts sharing a package must be on one `depends_on` path.
- `push-retry-not-halt` depends on this entry; `hardener-study-faults` and `keep-landed-run-records` come after it and touch teardown, catch-up and landing code in fabricengine.
- No peer run changes `internal/fabricengine`; `pr-modes-no-agent` edits `internal/testkit/envkit/envkit.go`, which D7 retargets, a text overlap only; `lym loom peers` lists no parent commit.

### Turn 2 — design batch (auto-picked)

Questions and picks, each with alternatives under Rejected alternatives: package boundary (D1), snapshot granularity (D2), mid-flow branching (D3), guards (D4), test migration and order (D6, D9), layout retarget without aliases (D7), close-out measure and target (D10), and the glossary clash of the entry's "plan" with the glossary's "plan" line (D11).

### Turn 3 — operator decision: fabric's tests test fabric alone

`ly:orch` relayed the operator (2026-11-26): "Fabricengine skal teste KUN fabricengine. Ikke noe annet."
About 25 of fabricengine's test files drive other modules, and 15 import configengine.
Recorded as D12; answered the parent.

### Turn 4 — operator decision: inverted dependencies

`ly:orch` relayed the operator: modules fabric tests drive sit above fabricengine; name for each of fabric's production imports of shedengine, shedrun, stencilstore, vscode and configengine whether it belongs or is cut.
A grep of what fabric uses from each showed run status and takeover (shedengine), run paths and locks (shedrun), stencil ordering (stencilstore), the workspace build (vscode) and config loading (configengine).
Recorded as D13: configengine stays, the other four are cut through general seams; answered the parent.

### Turn 5 — operator correction: real config in tests

`ly:orch` relayed the operator: hard-coded config values in tests go stale; tests use real config through configengine, which fabric's tests may import.
D12 dropped its config-as-options rule and allows configengine; answered the parent.

### Turn 6 — operator decision: no stand-in data for another module

`ly:orch` relayed the operator: no test uses fabricated stand-in data for another real module; a seam's tests feed it from the real module. Fabric's own hand-built state for its own rules is allowed. The parent files the PATTERN as its own note.
Recorded in D8, D12 and D13: seams speak fabric's vocabulary, adapters live in and are tested with the supplying module; answered the parent.

### Turn 7 — lens review

Every fork returned a report.

- bound-breaker 1 (BLOCKING, D12 import set too narrow for lymcwd, gitrepo, lock and kits): fixed; the set now bars only modules above fabric and D13's cut modules.
- bound-breaker 2 (D2 pre-lock refusals): fixed; D2 reads in phases, pre-lock included.
- bound-breaker 3 (D2 no-more-git bound broken by eager reads): fixed; later phases run only when the core asks, with a criterion.
- bound-breaker 4 and existing-paths 2 (detached children): fixed; D2 and D3 make each child a verb of its own with its own snapshot, the tag-before-delete order a core rule, with a criterion.
- bound-breaker 5 and walkthrough 4 (D10 target gameable by moving tests): fixed; D10 also records the whole run and the receiving packages' added time.
- bound-breaker 6, existing-paths 3 and interactions 3 (Mutations clause in `PATTERN-fabric-boundary`): fixed; D4 restates the background clause.
- bound-breaker 7 and interactions 6 (D1 gate-scope rationale overstated): fixed; narrowed to name-only importers.
- bound-breaker 8 (layout path functions and lymcwd): fixed; D1 says absolute paths and strings, lymcwd barred.
- bound-breaker 9 (envkit text overlap): fixed in Turn 1; no conflict.
- walkthrough 1 (live-pair probe moved to snapshot time): fixed; D4 keeps live-state probes as execution-time preconditions, with a criterion.
- walkthrough 2 (teardown seam has no action kind): fixed; D3 adds a seam-call action.
- walkthrough 3 (hubforge's imports keep fabric in other modules' scope): kept as D12's accepted fixture cost and raised to the parent under Open risks, since it bears on the operator's intent.
- walkthrough 5 (option types with engine seam fields): fixed; D1 keeps such types in the engine.
- existing-paths 1 (lymcwd owner maps and PATTERN backgrounds): fixed; D7 moves every scan, owner map and background entry with its symbol, `PATTERN-lym-files` listed.
- existing-paths 4 (stale scans): fixed by the same D7 rule.
- interactions 1 (D12 moves in the foundation need later tests): fixed; each concern part moves its own tests, guards land in part 7.
- interactions 2 (moved tests trip the vocabulary scan): fixed; D12 has moved tests drop bare side spellings.
- interactions 4 (lock of D3's re-reads): fixed; re-reads run under the lock held at that step today.
- interactions 5 (fallback test per verb): fixed; one fallback test.
- interactions 7 (D1 "every rule" against D4): fixed; D4's guards excepted.
- interactions 8 (tmux kit unnamed, kits for tier2): fixed; kits under `internal/testkit` allowed, tmuxkit named in Constraints.
- board-scope 1 (seam tests per verb are not "a few"): fixed; seam tests per flow shape, snapshot checks folded in.
- board-scope 2 (T11 dependency): kept under Open risks for the parent.
- leanness 1 (raddle order bullet unreviewable): fixed; rewritten as a per-part diff check.
- leanness 2 and 3 (repeated baseline and trim deliberation): fixed; Goal states the cause, shards fact dropped.
- leanness 4 (auto-mode bullets restate decisions): fixed; bare picks.
- leanness 5 (Constraints repeat D1 and D8): fixed; repeated bullets cut.
- leanness 6 (D12 module list goes stale): fixed; D12 names the rule, not the list.
- leanness 7 (Turn 2 repeats rejected alternatives): fixed; Turn 2 now points to them.
- leanness 8 (open risk deferring mid-flow re-reads): fixed; D3 makes every re-read a boundary or a phase, risk dropped.

## Rejected alternatives

- **Trim or fold more tests.** Tried in the 2026-11-03 to 11-09 speed work; 348 tests cut bought about 10 percent (`fabric-test-cuts-remeasure-2026-11-08.md`). The operator ruled it out.
- **More hub and pair templates.** Cuts fixture cost per test but keeps every rule behind a repository.
- **Sharding the package.** Measured in `fabricengine-shards-2026-11-09`: two and four shards were slower than one.
- **An in-memory git backend (go-git memory storage) behind the existing code.** Keeps rules coupled to git operations, behaves differently from the git CLI fabric runs (worktrees, hooks, push rejections), and would test the fake rather than the rule.
- **One core package holding names and rules.** Every importer of fabric's names would enter the gate scope of every rule change.
- **Refactoring inside fabricengine with no new package.** Nothing would stop a rule from calling git, and gate scope would not change.
- **A whole-hub snapshot, or one eager per-verb snapshot.** Adds git reads to verbs that read little and to refusals that stop early.
- **A detached child replaying an action list its parent computed.** The parent's snapshot is stale by the time the child acts on origin.
- **Core calling back into a git interface mid-decision.** Rules would again interleave with reads, and each test would need a fake git with call ordering.
- **Type aliases in fabricengine for moved names.** Callers would keep importing fabricengine; two names for one thing.
- **Seam tests per verb.** Around a hundred real-hub tests, against the entry's "a few".
- **Starting with teardown.** Biggest test count, but furthest from the snapshot shape.
- **Cross-module test moves all in the foundation part.** Their drops need the core and seam tests only later parts write.
- **Config fed to fabric tests as options instead of configengine.** The operator ruled it out: hard-coded values go stale.
- **Seams carrying another module's types, faked in fabric's tests.** The operator ruled out stand-in data for another module.
- **Binding `internal/fabriccli` by the import cut.** It wires the board, darn, webster and other engines into fabric's verbs; cutting those imports is a task of its own.
- **Building the detached children's seams in `internal/fabriccli`.** That would extend fabric's shed knowledge, which `PATTERN-fabric-boundary`'s isolation section forbids; the lym binary installs them at start instead.
- **Shed records as opaque bytes fabric copies without shed's codec.** The code-tip settle re-encodes the status it parses, so a raw copy could change the file's bytes.
- **Moving shed's record writes to a shed-side caller of fabric's general verbs.** The takeover write's commit, push and retry under fabric's records push lock would split across two modules.
- **The workspace passed in as bytes before the verb.** Fabric regenerates it after the verb's own changes, so no caller holds those bytes beforehand.
- **Judging D10's target against the board entry's 211 s.** That figure records no machine load, and load moves the package's time sixfold.
- **Core tests loading config through `internal/configengine`.** It needs fabric's spec, which the engine builds, and an initialized hub, and pulls `internal/lymcwd` and `os/exec` into tier0 core tests; engine tests keep loading real config, as the operator asked.

## Review rounds

### Round 1

Seventeen design and consistency findings fixed in the record; two decision findings (Open risks 1 and 3) went to the parent.
D12 now defines "above fabric" by dependency closure; D13 leaves `internal/fabriccli` outside the cut and has the lym binary install every seam at start,
carries shed's records as bytes with shed's codec, replaces the workspace bytes with an installed builder, and states required and optional seams.
D2 lists the teardown job's facts and verdicts and pins go-git's refusal of a reftable store; D3 and D4 place outcome classes, the wait action, mid-list failures, inverse actions and the recorded mutations;
D9 lands each PATTERN edit where it first holds; D10 measures before and after in one session.

## Question ledger

- Q1 (auto-pick): three packages, layout, core and engine (D1). Reason: tier0 isolation and a gate scope cut.
- Q2 (auto-pick): phased snapshot reads where today's code reads, detached children as their own verbs (D2). Reason: no added git processes, read-then-act guarantees kept.
- Q3 (auto-pick): action list ending at a branching action, seam-call action, engine loops (D3).
- Q4 (auto-pick): guards, live probes and locks stay in the engine (D4). Reason: the snapshot can be stale at execution.
- Q5 (auto-pick): no behaviour change, tests deleted only with their replacement (D5).
- Q6 (auto-pick): test migration rule, seam tests per flow shape (D6).
- Q7 (auto-pick): layout retarget, no aliases, scans move with symbols (D7).
- Q8 (auto-pick): shared builder kit (D8).
- Q9 (auto-pick): order foundation, push and origin, teardown, merge, pull, add, anchors and records (D9).
- Q10 (auto-pick, revised in review round 1): remeasure doc, a cut of at least half against a same-session before measure, and the whole-run measure (D10). Reason: seven in ten of the package's git processes were fixture git, and the 211 s baseline records no load.
- Q11 (auto-pick): "action list" over "plan" (D11). Reason: glossary line for "plan".
- Q13 (operator, via `ly:orch`): fabric's tests test fabric alone (D12); real config through configengine (correction); no stand-in data for another module. Placement of each kind of moved test is an auto-pick within it.
- Q14 (operator, via `ly:orch`): inverted dependencies cut (D13); configengine kept and where each adapter lives are auto-picks within it.
- Q12 (answered by the arbiter's added entry "push-retry-not-halt waits for the whole chain"): `push-retry-not-halt`'s dependency on this slug once it is a Sketch.
- Q15 (answered by the arbiter's added entry "hubforge, boardkit and envkit keep their import sets in this task"): whether D12 also asks for a slimmer `internal/hubforge`, and whether `internal/testkit/boardkit` and `internal/testkit/envkit` split off their name-only uses.
- Q16 (auto-pick, review round 1): `internal/fabriccli` outside the cut, seams installed at process start; shed records as bytes with shed's codec; a workspace builder seam; required and optional seams; outcome classes and a wait action; inverse actions with ownership tokens; core rows set config values as snapshot inputs while engine tests load real config (D3, D4, D12, D13).
