## Goal

When a tapestry run's Discussion is approved, lym copies the run's decision record onto the task's board entry and flags the entry `discussed`.
`lym loom decision add` keeps the stored copy current, `lym board get` shows the flag and prints the record, and a cancelled run leaves both on the entry.
This is the first half of the core of note `discussed-task-split`.

## Scope

In:

- A per-entry record file in the Bolt checkout beside `board.json`, and a `discussed` field in `board.json`, both written only by lym.
- The board's write, read, rename, removal and replay paths for the file and the field.
- Storing at Discussion's approval, through the Discussion-Bouncer row's approve seam.
- `lym loom decision add` refreshing the stored copy, and a `lym loom decision store` verb that re-copies it.
- `lym board get` printing the flag and, with `--record`, the record.

Out:

- Skipping Discussion for a discussed entry, the `cartoon` recipe and the sub-run block (task `shed-sub-run-block` and its follow-up).
- Any change to how tapestry or darn route: a discussed entry still runs its Discussion.
- Splitting a discussed entry; clearing the flag; showing the flag in the README or `lym board list`.
- Beam project records: pawl has no Discussion, and its project record stays in the beam's worktree.

## Decisions

### D1 — The record lives in `decision-<slug>.md` at the Bolt root, the flag in `board.json`

Each discussed entry's record is a file named `decision-<slug>.md` at the board directory's root, beside `board.json`, holding the run's decision record byte for byte.
`board.json` carries only a `discussed` boolean on the entry, stored as no key while false.
The file prefix and suffix are declared beside the design-file constants in `internal/boardengine/replay.go`.
The file is no render output: it never enters the render manifest, so a render never deletes it.
The board config load refuses a `design_prefix` equal to the record file's prefix, naming the way forward: set `design_prefix` in `board.yaml` to another value.

Rationale: the operator ruled the record out of `board.json` (2026-11-03), since a 10–30 KB record would swell the file and every replay of it.
A root file beside `board.json` rides the board's own Bolt commit and replay, as the design files do; the prefix refusal keeps the render from ever owning a record file.

### D2 — One board method stores the record and sets the flag, exempt from the run lock

A new Board method, `StoreDecisionRecord` (new), takes a slug and the record bytes and, in one board critical section, writes the record file atomically and then sets `discussed` on the entry.
The file is written before `board.json` is saved, so a crash between the two leaves a file without a flag and never a flag without a file; the next store overwrites the file.
It writes nothing, and spawns no sync, when the entry is already discussed and the file already holds the same bytes.
It refuses a slug the board holds no entry for, naming the way forward: restore the entry with `lym board upsert`, then re-run the step that stored.
It refuses a done entry, naming that a landed entry keeps the record it last stored, since the run ledger sets aside a board commit whose entries have all ended.
A reader method, `DecisionRecord` (new), returns the file's bytes, with found false unless the entry is flagged and the file exists.

`discussed` is no scope field: `scopeChanged` in `internal/boardengine/runlock.go` never compares it, as it never compares RunSettings, so a run-held entry takes the write, and the write stamps no revised date.
The record file is no entry field, so the run lock never sees it.
Bound: the store bypasses the run lock only for the flag and the file; every other field of a held entry stays locked, and only Go callers reach the method, since no board verb writes either (D3).

Rationale: the entry body states the lock exists so the orch does not change a task's text after its agents read it, and a record lym writes is not that; status is the precedent for a field lym sets past the lock.

### D3 — No board verb writes the flag or the record

An upsert, batch or merge payload may carry `discussed` only as the entry's stored value, false for a new entry, which is dropped as a no-op, as the `beam` and `front` keys are.
Any other value is refused, naming that the flag is never set by hand: lym sets it when a Discussion is approved, and `lym loom decision store` re-copies the record of a run past its approval.
No `lym board` verb writes a record file.

Rationale: the entry body requires lym itself to write both; a hand-set flag with no record would break the invariant D2 keeps for the follow-up's gate.

### D4 — The record file follows its entry through rename and removal

- A rename moves the entry's record file to the new slug's name in the same write.
- A write whose net change removes an entry, judged as the run lock judges it, removes its record file after the save, under the board lock: a remove, a prune, or a merge that removes a slug it does not upsert again.
- A status-only write leaves both: `lym batten cancel` clears the status and so keeps the flag and the file, and so does a landing that marks the entry done until a prune removes it.

Rationale: the record belongs to the entry, and an orphaned file would read as a record of a later entry reusing the slug.

### D5 — The board replay carries the file and the flag

`FabricReplay` claims `decision-<slug>.md` files at the Bolt root, and `CommitRuns` names the slug of each record file a commit touches.
`discussed` joins the replay's field table, merged by entry like status.
A record file replays by the design-file rule: the commit's side is applied when origin's file is still as the merge base or the commit's parent had it, and otherwise origin's file is kept and the local one reported lost.
A commit that writes a record file whose entry origin's board does not hold after the entry changes is reported lost and writes nothing; a commit that removes a record file follows the design-file rule alone, so a replayed removal or rename never leaves the old file on origin.

Rationale: the operator required the file be "carried by the board's own commit and replay"; the design-file rule is the existing by-file merge, and checking the entry only on a write stops an orphan without blocking the removal D4 makes.

### D6 — Discussion-Bouncer's approve seam stores the record

The Discussion-Bouncer row in `contracts/recipes/tapestry-recipe.yaml` gains `approve_seam: discussion`.
The Bouncer entry in `internal/shedrecipe/entries_bouncer.go` accepts `discussion` beside `plan` and resolves it to a new Env closure, `ApproveDiscussion` (new), guarded by `requireSeam` as `plan` is.
loomcli wires it to read the run's decision record working copy and call `StoreDecisionRecord` for the run's board slug, the one `internal/boardengine#Board.SetRunStatus` is called with.
The Bouncer calls its approve seam on every approving path: a judged approval, an accepted round, a decision accept, a skipped review and a zero-round settle.
On each path the seam runs before the commit seam, and on a path that writes a carry-over, after it, so the stored bytes are the bytes the approval commit lands.
A failed store fails the settle as a failed `ApprovePlan` does: the run halts at Discussion-Bouncer with the error, and a resume re-settles and stores again.
Bound: a `lym loom goto` that jumps past Discussion-Bouncer skips the store; `lym loom decision store` (D9) is the way to store such a run's record.

Rationale: the approve seam is the Bouncer's existing hook for "the segment approved", reached on every approving path, and it adds no recipe row, so the row list, the progress numbers and the rows `cartoon` copies from tapestry stay as they are.

### D7 — The discussion approval row is the Bouncer a DiscussionWrite row hands to

A recipe's discussion approval row is the Bouncer row named by the `on_done` of its row whose engine is DiscussionWrite.
A recipe test asserts, over every embedded loom recipe, that each discussion approval row carries `approve_seam: discussion` and that no other Bouncer row does.
D8 and D9 find the approval row by the same rule.

Rationale: task `shed-sub-run-block` is building `cartoon` from tapestry's Discussion rows in parallel; whichever task lands second fails this test until its copy of the row carries the key, so the store cannot be lost between the two.
`artifact_paths` cannot select the row, since every Code-Bouncer names the decision record too.

### D8 — `lym loom decision add` refreshes the stored copy

After its commit succeeds, `decision add` copies the committed record onto the board entry through `StoreDecisionRecord`, when the run has passed its Discussion approval.
A run has passed it when its recipe has a discussion approval row (D7) and the run's current row comes after every row of that row's review segment in the recipe's row order, judged by the recipe's rows, never its name.
So a darn run, a sampler run and a beam's project record are never copied, and a run sent back into Discussion is not copied until its next approval stores the record.
The copy is the whole committed record, so carry-over blocks later review segments wrote into the record reach the board copy too.
A failed copy never fails the verb, since the entry is already committed: it is logged at Warn and reported on the success envelope as `record_store_error`, naming `lym loom decision store` as the way forward.
Bound: a call landing while Discussion-Bouncer is settling is not copied, and the approval may store the record without it; `lym loom decision store` closes that window by hand.

Rationale: the entry body requires decision add to update the stored record "the same way", so the whole committed record is copied, as at approval; copying only past the approval keeps an unapproved record of a second run off an entry an earlier run already flagged.

### D9 — `lym loom decision store` re-copies an approved record

A new verb, `lym loom decision store [<slug>]`, copies the run's committed decision record onto its board entry through `StoreDecisionRecord`.
It admits the run's parent orch, the operator's own unnamed shell and the run's own arbiter strand, each judged as `decision add` judges that caller, and refuses every other named session.
It is refused, naming the way forward, on a recipe with no discussion approval row and on a run that has not passed its approval as D8 judges it.
It changes nothing in the run: no commit, no routing and no notice.
Bound: it stores only the run's own committed record, and only past the approval, so it can flag an entry discussed only where the run's approved record exists; a goto past Discussion-Bouncer by the parent counts as past, which the parent's own goto decided.

Rationale: D6 and D8 each leave a case where the board copy is missing or stale, and a re-run of `decision add` would append a duplicate entry; one idempotent verb over the same store call is the one way forward both name.

### D10 — `lym board get` shows the flag and prints the record

The `get` JSON envelope carries `discussed` as a field of the entry, present only when true.
`lym board get <selector> --record` writes the stored record alone, verbatim, as `--body` writes the body.
It is an error, naming the way forward, when the entry does not exist, when it is not discussed, and when `--body` is passed beside it.

Rationale: the entry body asks that `lym board get` show the flag and can print the record; `--body` is the existing shape for printing one field verbatim.

## Constraints

- `PATTERN-tolerant-records`: `discussed` is a declared `Task` field; an older binary keeps it as an unknown key and keeps writing it back.
- `PATTERN-named-constants`: the record file's prefix and suffix are declared once in `boardengine`.
- `PATTERN-loom-contract`: D8 and D9 decide by the recipe's rows, the discussion approval row of D7, never by the recipe's name.
- `PATTERN-shed-recipe-registry`: D6 adds no producer and no row; it extends the Bouncer entry's existing `approve_seam` key.
- `PATTERN-refusal-way-forward`: every refusal D1, D2, D3, D9 and D10 add names its way forward, and a test reaches each.
- No PATTERN entry is added, changed or removed.

## Auto-mode assumptions

- Runs already past Discussion when this lands are not backfilled automatically; `lym loom decision store` stores one by hand.
- Nothing clears `discussed` in this task: a later Discussion of the same entry overwrites the record and keeps the flag.
- An older binary's remove, prune or rename leaves a `decision-<slug>.md` file behind, and its replayer does not carry one; the orphan is harmless, since `DecisionRecord` requires the flag, until every machine runs the new binary.

## Open risks

- Task `shed-sub-run-block` keeps tapestry unchanged while it builds `cartoon` from tapestry's Discussion rows, and this task adds `approve_seam: discussion` to tapestry's Discussion-Bouncer: the parent should confirm that D7's guard, rather than a hand merge or a different owner of the row, is the agreed coordination.

## Acceptance criteria

- D1: a stored record lands in `decision-<slug>.md` at the board root, byte for byte, `board.json` holds `discussed: true` and no record text, and a later render leaves the file in place; a `design_prefix` equal to the record prefix is refused with its way forward.
- D2: `StoreDecisionRecord` on a run-held entry passes the run lock and changes neither its revised date nor any other field; a repeat with the same bytes writes nothing; an absent slug and a done entry are refused with their ways forward.
- D2: `DecisionRecord` answers not found for a file without the flag.
- D3: an upsert, batch or merge payload setting `discussed` to a value other than the stored one is refused with its way forward, and one carrying the stored value passes.
- D4: a rename moves the record file, a remove, a prune and a merge each remove the removed entry's file, a merge that upserts the slug it removes keeps the file, and a `lym batten cancel` leaves the flag and the file.
- D5: a replayed commit that adds, changes or removes a record file applies it when origin's file is untouched and reports it lost when origin changed the file too; a written record file whose entry origin lacks is reported lost; a replayed entry removal and a replayed rename leave no old record file on origin; `discussed` merges by entry and `CommitRuns` names a record file's slug.
- D6: an approved Discussion-Review, by a judged round, an accept and a zero-round settle, leaves the approved record on the entry and the entry flagged; a failing store halts the run at Discussion-Bouncer, and a resume stores it.
- D7: the guard test fails for a recipe whose discussion approval row lacks `approve_seam: discussion` and for one whose Code-Bouncer carries it.
- D8: `decision add` on a tapestry run past its approval leaves the entry's stored record equal to the committed record with the new entry; on a run inside Discussion-Review, on a darn run and on a beam's project record it leaves the board untouched; a failed copy still exits ok and reports `record_store_error`.
- D9: `lym loom decision store` copies the committed record past the approval, is refused with its way forward on a darn run and on a run still inside Discussion-Review, and refuses a named task session.
- D10: `lym board get` shows `discussed: true` on a discussed entry and omits it otherwise; `--record` prints the record verbatim, and is refused on an undiscussed entry, an absent entry and beside `--body`.

## Notes for the plan writer

- The board's `afterSave` hook in `boardCriticalSection` is the existing place for a file step under the board lock after the save, used by the legacy retire; D4's removals fit it, while D2 writes its file inside the mutation, before the save.
- `ApprovePlan` in `internal/loomcli/wiring.go` is the wiring precedent for D6's closure, and `boardengine.OpenHub` reaches the board from the task worktree as the run status write does.
- `checkDarnRecord` and `committedLacksEntry` in `internal/loomcli/decision.go` already read the committed record; D8's copy belongs after the existing post-commit check, before the notices.
