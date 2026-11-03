# Store the approved decision record on the board entry, flagged discussed

An approved Discussion now copies the run's decision record onto the task's board entry and flags the entry `discussed`.
A cancelled run leaves the record and the flag on the entry, so a new task can take the record along.
This is the first half of the core of note `discussed-task-split`; skipping Discussion for a discussed entry stays with task `shed-sub-run-block` and its follow-up.

## What changes

- The record lives in `decision-<slug>.md` at the Bolt root beside `board.json`, byte for byte.
  `board.json` gains only a `discussed` boolean, stored as no key while false, so a 10–30 KB record never swells the file or its replays.
- `Board.StoreDecisionRecord` writes the file atomically and then sets the flag in one critical section.
  It is exempt from the run lock for those two writes only: `scopeChanged` never compares `discussed`, and no revised date is stamped.
  `Board.DecisionRecord` reads the file back and reports found only for a flagged entry whose file exists.
- Discussion-Bouncer in `contracts/recipes/tapestry-recipe.yaml` gains `approve_seam: discussion`.
  The Bouncer entry in `internal/shedrecipe` resolves it to a new `ApproveDiscussion` closure, wired in loomcli to store the working-copy record for the run's board slug.
  The seam runs on every approving path, before the commit seam.
- `lym loom decision add` copies the committed record onto the entry once the run is past its Discussion approval.
- A new verb, `lym loom decision store [<slug>]`, re-copies the committed record by hand.
- `lym board get` shows `discussed` in its envelope; `--record` prints the stored record verbatim.
- The board follows the file through its write paths: a rename moves it, a remove, prune or merge that removes an entry deletes it, and replay claims and merges record files like design files.

## Design decisions

- No `lym board` verb writes the flag or the record.
  An upsert, batch or merge payload may carry `discussed` only as the stored value; any other value is refused with the way forward.
- The board config load refuses a `design_prefix` that is a prefix of `decision-` (the empty one included) or starts with `decision-`, so a render can never own a record file.
- The "discussion approval row" is the Bouncer row named by the `on_done` of the row whose engine is DiscussionWrite.
  A recipe test asserts every embedded loom recipe carries `approve_seam: discussion` on that row and on no other Bouncer row, so a recipe built from tapestry's Discussion rows by `shed-sub-run-block` fails the test until its copy carries the key.
- `decision add` and `decision store` judge "past the approval" by the recipe's rows, never its name, so darn runs, sampler runs and a beam's project record are never copied.
  A run moved past Discussion-Bouncer by `lym loom goto` counts as past.
- A copy failure after `decision add` commits never fails the verb: it is logged at Warn and reported as `record_store_error`.
- At approval, a store refused because the entry is done logs a warning and the settle goes on; any other store failure halts the run at Discussion-Bouncer.
  On an accepted round the accept settles before the seam, so a failed store there is not retried by a resume; the next approval of the segment stores the record.
- A rename onto an orphan `decision-<new>.md`, when no file exists at the old name, removes the orphan so the renamed entry cannot serve it as its record.
- Runs already past Discussion when this lands are not backfilled; `lym loom decision store` stores one by hand.
- An older binary's remove, prune or rename leaves an orphan `decision-<slug>.md` or a flag without a file; `DecisionRecord` answers not found for both until the next approval stores the record.
- Approval inside a sub-run block is out of scope: a run of the `shed-sub-run-block` recipe stores no record until the follow-up that adds the gate on the cartoon block.

## To check by hand

- The `cmd/lym` package failed the full verify once with no changed package named; the fix commit `d4633f36` shortened the board upsert and decision add help pages under the Long cap. Run `lym gate test ./cmd/lym` to confirm it is clean.
- `boardcli` store verbs: a plan finding flagged the store verbs as reaching the wrong symbol (scope, blocking). Confirm `decision store` in `internal/loomcli` and the `--record` path in `internal/boardcli` call `StoreDecisionRecord` and `DecisionRecord` as the design says.
- A removal after the save: a plan finding questioned how a record-file removal error after the save is reported. Check that a failed removal in the board's `afterSave` step surfaces as the write's error without undoing the saved removal.
- Confirm with the parent that the recipe test is the agreed coordination with `shed-sub-run-block` for the Discussion-Bouncer row.
