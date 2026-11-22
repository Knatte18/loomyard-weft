---
verdict: BLOCKING
findings:
  - id: F8
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:42
    summary: "D2's main path rests on a false premise: it asks the live writer to write `_lym/move-note.md`, but every writer D2 names is barred from writing outside its own output files. The Discussion stencil makes everything but its three files read-only (`_bolt/_lym/stencils/loom/loom-template-discussion.md:156-174`), and the Webster master never writes outside its two contract files (`_bolt/_lym/stencils/webster/webster-template-master.md:306`). The parent directive has a session answer an instruction outside its limits by naming the limit (`_bolt/_lym/stencils/parent/parent-directive-parent.md:22`). So the ask returns a refusal, not a note, and D3 covers only a writer that is not live, so the procedure has no branch for it."
    fix: "In D2, ask the live writer by `lym creel post --expect-reply <strand>` to answer with the note's first three parts in its reply text, writing no file. The orch writes the move note itself in its own `.scratch/` from that reply, then holds the run and adds the fourth part. Drop `_lym/move-note.md` in the task worktree and the 'copies the note to its own `.scratch/`' step, and rewrite D2's sentence on the uncommitted note and its Rationale to match: the note never sits in the task worktree, and the board copy of D4 stays the durable one. Add to D3 that a writer that refuses, or replies without the three parts, is handled as no live writer. Update the Auto-mode assumptions line naming `_lym/move-note.md` and the D2 acceptance bullet, which then reads: ask the live writer by creel, while no pause covers the slug, for the first three parts in its reply; write the note in the orch's `.scratch/`; hold with `lym tender pause <slug>`; add the tip SHA and merge base read after the hold."
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    carries: F5
    location: _lym/discussion/decision-record.md:90
    summary: "D4 step 3's test still mixes refusals into partial finishes. The hub-pause refusal, raised before anything ends, reads 'run \"lym tender resume\", then \"lym batten cancel <slug> --remove\" again' (`internal/battencli/cancel.go:303-304`). It names the cancel again, so D4 files it as a partial finish, and the orch takes the fix: a hub-wide `lym tender resume`. That lifts every pause entry, the operator's and D2's slug hold alike (`internal/tendercli/cli.go:194`). The run then commits again, and the cancel archives a tip other than the recorded one, whose tag step 3 then waits for forever. The owned-elsewhere refusal names both `lym adopt` and the cancel on the owner (`cancel.go:715`), so it fits both classes."
    fix: "Rewrite D4 step 3's two cases. A partial finish is an ok envelope with `end_only: true` or a non-empty `way_forward`, or an error whose way forward is 'fix the cause …, then run … again' after the run's processes ended. A refusal is every error the cancel raises before it ends anything: the hub-pause refusal, the owned-elsewhere refusal, 'the work … reached its parent', 'landed by another machine', an origin-stopped refusal and a nothing-left refusal. D7 applies to every refusal. Add that the orch never runs `lym tender resume` without a slug: on the hub-pause refusal it stops with its slug pause in place and tells the operator. Update D7's pointer and the D4 acceptance bullet to match."
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    carries: F2
    location: _lym/discussion/decision-record.md:96
    summary: "Steps 3 and 4 still treat the code tag as the end of the detached teardown. For a discussed entry, step 4's `lym board upsert --record-file` is refused while the slug's records branch reads live on origin (`internal/boardengine/recordupsert.go:38-46`, through `internal/fabricengine/remoteleftover.go:86-114`). The wait for that branch to go sits in step 5, after the upsert. The tag wait has no bound when the teardown child dies without a notice. The owner Constraint (line 166-167) says the fresh run 'needs nothing local to the old machine' once the tag is on origin. But a plain fetch never auto-follows an archive tag whose branch was deleted, and the code store keeps git's default refspecs (`pattern/PATTERN-single-repo-hub.md`), so on another machine D6's diff finds no tag."
    fix: "In D4 step 3, make the orch confirm both sides before step 4: the code tag through `git -C <prime> ls-remote origin 'refs/tags/archive/<slug>/*'`, and no records branch of the slug through `git -C <records repo> ls-remote origin`. Move that condition out of step 5, which keeps only the pruning fetch. Bound the wait: when neither the tag shows nor a remote-teardown notice arrives within a stated time, such as ten minutes, push the tag by hand with the command the notice would name, `git -C <prime> push origin refs/tags/archive/<slug>/<12 hex>`, then check again. In D4 step 4's section contents and in D6, name the fetch the fresh run makes before the diff, `git fetch origin refs/tags/archive/<slug>/<12 hex>:refs/tags/archive/<slug>/<12 hex>`. Reword the owner Constraint to say the fresh run fetches that tag from origin. Name both in the D4 and D6 acceptance bullets."
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    carries: F3
    location: _lym/discussion/decision-record.md:83
    summary: "D4 step 2 checks the new parent 'with `lym board get`', but the `lym beam ready` flag is no board field. It is a ready record, `beam-ready.json`, committed in the beam worktree's `_lym/landing` directory (`internal/beamcli/cli.go:916-921`, `internal/loomengine/config.go:106-114`). So a beam at Beam-Await that is flagged ready passes the check as written, and the moved member never starts (`contracts/orchrole/role-beams.md:61`). The check also runs before the cancel and the tag wait, so a beam that moves past its hold-free rows in between makes step 4's `project` write refuse after the cancel, with no path for that case."
    fix: "In D4 step 2, name the read of the ready flag next to `lym board get`: a beam qualifies only when its worktree's `_lym/landing/beam-ready.json` is absent. In D4 step 4, add that a refused `project` write leaves the entry a draft under its old parent with the move note written in, and that the orch tells the operator. Update the D4 acceptance bullet to name the ready-record read."
prior:
  - id: F1
    status: resolved
    answer: "D2 now asks first, then holds with `lym tender pause <slug>`, pinned at the cited lines; D7 lifts with `lym tender resume <slug>`."
  - id: F2
    status: open
    answer: "The code tag is now named from the recorded tip and confirmed by ls-remote. The same detached teardown still outruns step 4 on the records side, the wait is unbounded, and the owner bullet's 'needs nothing local' still fails without a tag fetch; carried as F10."
  - id: F3
    status: open
    answer: "The check exists, but its ready-beam clause names a state `lym board get` cannot show; carried as F11."
  - id: F4
    status: resolved
    answer: D4 step 5 runs the pruning fetch while stale-records-ref-blocks-rerun has not landed.
  - id: F5
    status: open
    answer: "The two cases are now defined by output, but the definition files the hub-pause and owned-elsewhere refusals as partial finishes; carried as F9."
  - id: F6
    status: resolved
    answer: "The stored-record carrier is an H3 under `## Notes for the plan writer`, its parts never H2."
  - id: F7
    status: resolved
    answer: D8's glossary line reads as proposed.
---

### [BLOCKING] D2 asks a writer to write a file its role forbids

**Location:** `_lym/discussion/decision-record.md:42`
**Issue:** D2's main path has the live writer write `_lym/move-note.md` in the task worktree.
Each writer D2 names is limited to its own output files: the Discussion writer to its three files, and the Webster master to its two contract files.
The parent directive rendered into every spawned role has a session answer an instruction outside its limits by naming the limit.
The orch's ask therefore comes back as a refusal.
D3 triggers only when no writer is live, so a live writer that refuses falls through both decisions.
D2 was rewritten in round 1 (F1), so this sits on a changed hunk.
Three lenses found it independently, and I confirmed the Discussion stencil's limit myself.

### [MEDIUM] Partial-finish test files two refusals as partial finishes

**Location:** `_lym/discussion/decision-record.md:90`
**Issue:** Carries F5.
The test 'an error whose way forward names the cancel itself to run again' also matches refusals raised before the cancel ends anything.
The hub-pause case is the harmful one.
Taking its fix runs a hub-wide `lym tender resume`, which lifts the operator's pause and D2's slug hold together.
The run moves on, and the tag step 3 waits for is named from a tip the cancel no longer archives.
The owned-elsewhere refusal matches both definitions.

### [MEDIUM] Teardown wait covers the code tag alone, without bound or fetch

**Location:** `_lym/discussion/decision-record.md:96`
**Issue:** Carries F2.
The F2 fix pinned the code tag, but the cancel's remote half still outruns the procedure in three ways.
First, the discussed-entry upsert of step 4 is refused while the records branch still reads live on origin.
Second, the tag wait has no bound when the teardown child dies silently.
Third, the owner Constraint promises a fresh run on another machine needs nothing local, yet no step fetches the archive tag, which plain fetch does not follow.

### [MEDIUM] New-parent check cannot see the beam ready flag

**Location:** `_lym/discussion/decision-record.md:83`
**Issue:** Carries F3.
The check added in round 1 reads the board, but `lym beam ready` records its flag in the beam worktree, not on the board.
The window between the check and the `project` write is also open, with no stated path when the write refuses after the cancel.

## Focus answers

- D2, D4 step 1, D7 hold: `lym tender pause <slug>` is the right hold, the ask before it is delivered, and D7's `lym tender resume <slug>` lifts the same entry.
  What fails is what the ask asks for (F8).
- D4 step 2 and the `--remove` Constraint: the tags reach origin from the detached child after the cancel returns, and the code tag's name is derivable from the recorded tip and confirmed by `ls-remote`.
  The records side is not confirmed before the board write (F10).
- D4 ordering: the check exists, but it misses the ready flag (F11).
- D4 step 3, discussed entry: the H3 `### Move note` under `## Notes for the plan writer`, added last when absent, keeps the stored record's fixed H2 set and order from the discussion spec.
  Not a defect.
- D4 step 3 and D7, partial finish versus refusal: not told apart by the actual output for the hub-pause and owned-elsewhere refusals (F9).

## Rejected

- The parent check racing a beam past its hold-free rows (interactions I3): folded into F11 rather than kept alone.
- The diverged-tips case of D4 step 1 (bound-breaker item 4): both tips are tagged and the worktree tip's tag is pushed, so the recorded tag exists; no defect.
