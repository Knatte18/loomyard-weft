# Support log — orch-continue

## Interview

### Turn 1 — scope and default (autonomous)

The board entry offers two shapes: a `--continue` flag, or making continue the default for a dead strand.
Recommendation: an explicit flag, since a dead or absent strand also follows `lyx orch stop`, a crash or a wish for a clean context.
Auto-picked the flag.

### Turn 2 — which id to record (autonomous)

Exploration showed clear mode starts a new provider session on `/clear`, and the session-start signal names the new id; `internal/orchengine/doc.go` records the same fact in its reed-rebirth residual.
Recommendation: record the current id, kept current by the watcher from the newest session signal, not the launch id.
Auto-picked.

### Turn 3 — where the record lives (autonomous)

Options: orch state written by `start` and the watcher, or read the dead strand's shuttle run at `start --continue` time.
Recommendation: orch state, because `lyx orch stop` and a relaunch remove the strand, and the run may be gone with it.
Auto-picked.

### Turn 4 — launch path and surface (autonomous)

Recommendation: `--continue` reuses the adopt branch wholesale, with its guards, and the envelope adds `session_id`; `lyx orch status` shows the recorded id.
Auto-picked both.

### Turn 5 — lens review

- premises MEDIUM, the watcher does not read session signals every tick: fixed, D4 makes the per-tick files-only reading explicit and the plan note says it is new.
- premises MEDIUM, `start` cannot see a plain launch's minted id: fixed, D4 has shuttle's start seam return the id.
- premises LOW, `/clear` minting a new id is not verified live: rejected, Open risks already covers it.
- premises LOW, D2 holds: no change needed.
- board-fit MEDIUM, overlap with the running `creel-stage1-fixes` in shuttle: fixed, Constraints names the overlap and that the change is additive; `depends_on` is the orch's call.
- board-fit LOW, `creel-notices` may drop the id tracking: rejected, the tracking sits in the watcher state that note keeps.
- board-fit LOW, per-role records under `orch-split`: rejected, it designs for a note not yet built.
- board-fit LOW, prime-bound wording against `orch-any-worktree`: fixed, wording names the orch's state directory and working directory.
- leanness MEDIUM, Auto-mode assumptions copy the Decisions: fixed, reduced to a pointer.
- leanness LOW, D1 and D4 rationales argue against rejected alternatives: fixed, rephrased as positive rationale.
- leanness LOW, generic cobra and test-rule constraint bullets: fixed, cut; the test-budget line stays.
- leanness LOW, boot autostart exclusion guards nothing: fixed, cut.
- leanness LOW, plan notes repeat `relevant.md`: fixed, only the non-obvious hint stays.
- goal-scope MEDIUM, a plain start after reboot silently loses the conversation: fixed, D1 adds the envelope hint naming `--continue`; a later round moved the hint to `--adopt` with the previous id.
- goal-scope LOW, interaction with `lyx reed resume`: fixed, the Out bullet names the recovery.
- goal-scope LOW, envelope field and status line go past the ask: rejected, both serve the entry's stated pain.
- goal-scope LOW, Goal says "any death" though D1 needs a request: fixed, Goal reworded.
- pattern-fit: no departures; its four LOW confirmations need no change.
- scenarios MEDIUM, a quiet tick could regress the id to the launch id: fixed, D4 folds over all the run's signals, with a must-cover criterion.
- scenarios MEDIUM, a session with no transcript refuses with a misleading message: fixed, D3 appends the two ways forward to a launch error under `--continue`, with a criterion.
- scenarios MEDIUM, a failed launch after corpse removal: fixed, D3 and a criterion state the recorded id survives.
- scenarios LOW, watcher down while the strand lives: fixed, added to Open risks.
- scenarios LOW, compact mode not pinned: rejected, the unchanged-id criterion covers it.
- scenarios LOW, a fork's hook line could carry a foreign id: rejected, Claude Code sends the parent session id there today.
- scenarios LOW, the attack-surface bound holds: no change needed.

## Rejected alternatives

- `--continue` as the default for a dead strand: would resume a crashed or deliberately stopped session where a fresh one is wanted, and changes the start contract.
- Telling a dead strand (continue) from an absent one (fresh) by default: subtle, and a reboot's strand state is not a reliable signal of intent.
- Recording only the launch id: stale after the first `/clear` cycle in clear mode.
- Reading the session id from the dead strand's run at start time: lost after `lyx orch stop` or a corpse removal.
- A new `continue` prompt source and stencil: duplicates the adopt prompt with no difference in what the session needs to read.
- Fixing reed's resume-after-rebirth replay of the launch id: a reed change outside this entry's ask.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Default or flag? Resolved: explicit `--continue` flag (auto-pick).
- Q2 Launch id or current id? Resolved: current id, tracked by the watcher (auto-pick).
- Q3 Record in orch state or read from the dead run? Resolved: orch state (auto-pick).
- Q4 Separate launch path or adopt reuse? Resolved: adopt reuse, envelope adds `session_id` (auto-pick).
- Q5 Show the id in `lyx orch status`? Resolved: yes (auto-pick).
- Q6 Fix reed's rebirth replay of the launch id? Resolved: out of scope (auto-pick).
- Q7 Should a fresh launch point back at the previous conversation? Resolved: yes, through the envelope hint naming `lyx orch stop` then `lyx orch start --adopt <previous id>` with the previous id inline, since a `--continue` after the launch would resume the new session; from the goal-scope lens (auto-pick).
