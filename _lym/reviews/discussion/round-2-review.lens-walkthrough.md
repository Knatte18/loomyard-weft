```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism end to end on one concrete case.

**Decisions walked**
- D1: jacquard run alone of a Sketch part. Preflight → Record-Seed seeds the stored record → Drift-Baseline … Friction-Reflect. Purpose met.
- D2: new tapestry run of a discussed entry. Motif's `undiscussed` gate seeds the record and support log and commits (`internal/loomcli/discussionrecord.go:59-84`). It skips to SkipTo Jacquard, then the Jacquard block holds and enters Preflight. Record-Seed finds both files and writes nothing. Purpose met.
- D2: in-flight run parked at Discussion-Burler-Review. Discussion-Bouncer's empty `on_done` now goes to the block exit Jacquard → Preflight → Record-Seed (no-op) → Drift-Baseline. Purpose met.
- D2: in-flight undiscussed run parked at old Preflight. Preflight → Record-Seed is Stuck → goto Discussion-Write → Discussion → Jacquard → Preflight → Record-Seed (no-op). Purpose met.
- D3: Expand over tapestry's two block rows.
  - The Motif row gets SkipTo Jacquard.
  - The Jacquard row's exit is empty, so `inline` passes exit "" and Friction-Reflect keeps its empty `on_done` (`expand.go:85-92`, `:158`).
  - The Jacquard row gets OnDone Preflight and SkipTo "", and the `always` gate never skips (`producer.go:132-137`).
  
  Purpose met.
- D4 contract: expanded tapestry's `on_done` path runs Motif → Discussion-Write → Discussion-Bouncer → Jacquard → Preflight → Record-Seed → … → build row. The seed row follows the record row, and Finalize is followed only by terminals. Purpose met.
- D4 decision verbs: jacquard run at Record-Seed with the record missing gets the resume way forward. Past Record-Seed with the record uncommitted, it gets `goto --to Record-Seed` and a commit on resume. Purpose met.
- D4 Stuck, jacquard run in a batten pair: cancel --remove → upsert tapestry → release → batten run gives a fresh tapestry run that runs Discussion. Purpose met.
- D4 Stuck, jacquard run in a non-batten pair: purpose not met. Finding 1.
- D5 batten site, jacquard entry with no record and no status file: refused before Worktree-Create. Purpose met.
- D5 batten site over a done motif run: purpose not met when the motif pair left its task branch. Finding 2.
- D5 `lym loom start` site: batten's Run-Shed starts the child with `lym loom start` (`internal/battencli/wire.go:1543`), and only that verb seeds loom's status file (`internal/battencli/arm.go:750`). So the first-start check fires in a batten pair, and both of its ways forward clear it. Purpose met.
- D5 first start through `lym loom step`: purpose not met. Finding 3.
- D6: a filed jacquard part carries the discussed flag and record. Batten's guard passes, D7 gives the profile its jacquard block, and Record-Seed seeds the record. Purpose met.
- D7, hub that removed `solid`: the follow adds `defaults.jacquard: solid` (base and file both lack it, `follow.go:173-181`). The reinterpretations run after the follow in the same walk (`internal/hubreconcile/walk.go:250-266`) and drop that default. Purpose met.
- D7, profile the operator added: the follow leaves it alone, and the reinterpretation adds its jacquard block. Purpose met.
- D8 and D9: a jacquard run's own usage record keys `jacquard`, and a discussed tapestry run is re-keyed to jacquard. Purpose met.
- D10 and D11: no mechanism to walk.

**Findings**

1. MEDIUM, design, `_lym/discussion/decision-record.md:80`
   - Summary: D4's jacquard way forward from Record-Seed's Stuck is batten-only, but D5 hands the non-batten pair's lost-record case to this same Stuck.
   - Walk: D5 (line 117-118) says a re-run of `lym loom start` over an existing status file passes, and that Record-Seed's Stuck guards a run whose entry lost its record. D5 (line 115) splits a non-batten pair's way forward: remove the seed, re-seed with `lym shed seed self --recipe tapestry`, start again. D4's Stuck names only `lym batten cancel &lt;slug&gt; --remove` … `lym batten run &lt;slug&gt;`. In a pair batten did not make, batten holds no run to cancel, so the run stays Stuck.
   - Evidence: `decision-record.md:78-81` against `:113-118`; `sharedbootstrap.go:128-131`, where bootstrap seeds through `loomshed.Seed` and a re-entry passes as ErrSeedExists.

2. MEDIUM, design, `_lym/discussion/decision-record.md:108`
   - Summary: following the batten-site way forward over a done motif run can end in Worktree-Create's leftover-branch halt, not a running start.
   - Walk: D5 names `lym batten cleanup --apply`, then the upsert to tapestry. `Cleanup` never touches a pair or a branch (`cleanup.go:111-116`). `doneSlugRefusal` states that a torn-down pair keeps its task branch locally and on the remote, and itself names deleting it (`arm.go:940-948`). The tapestry start passes the guard, since `leftPartNames` does not name a task branch (`startguard.go:443-445`). It then halts at Worktree-Create on ErrBranchExists, whose way forward is `lym batten cancel --remove` (`internal/battencli/wire.go:375-394`).
   - The jacquard clear has the same open question: `ClearMotifRun` discards no branch (`cleanup.go:137-142`), and the record's own plan-writer note leaves that unverified.

3. LOW, design, `_lym/discussion/decision-record.md:110`
   - Summary: a first start in a worktree through `lym loom step` passes D5's refusal, because the check is placed only in `lym loom start`.
   - Walk: `step` calls `seedAndCommitBootstrap` too and seeds the absent status file (`sharedbootstrap.go:65`, `:128`). So a jacquard seed of an entry with no record runs to Record-Seed's Stuck instead of being refused at start. That Stuck's batten-only way forward is finding 1.

Tool calls:
- grep for `SeedDiscussedRecord`, the batten child's start verb and the run-verdict wiring: the context lacked the gate's seam, how batten starts its loom child, and where `runVerdict` reads from.
- Read `internal/loomcli/discussionrecord.go` 30-86: to learn whether the Motif gate's seed writes the support log, for D2's "Record-Seed writes nothing".
- grep for `hubRunVerdict` and the board's done status: to check whether a start after cleanup is refused by the run verdict.
