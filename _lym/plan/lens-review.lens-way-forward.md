```yaml
lens: way-forward
outcome: reported
```

Way-forward lens on the raddle plan: I traced each refusal, stop, reset and recovery step the cards add to the next step it names, and checked whether that step can succeed. Defects only.

**BLOCKING — Card 13 (D2), the promoted Sketch is not released.**
- Card 13 says the task `PromoteTo(…, "raddle")` produces "is released (empty status), so its parent's dobby starts the run". It names no way past the draft rule.
- `applyDraftRule` (`internal/boardengine/draft.go:37-58`) drafts every entry that was not a task in `loaded`. Its comment says "a note or project made a task … read draft", so the promotion lands as `draft`.
- The opt-out comes only in card 16 (`applyDraftRule` gains a `released` list fed by a `raddleFiled` result), which is a later card and only for `FileRaddle`.
- Result: D2's criterion ("makes a Sketch a released task … in one write") fails, and `TestDiscussedPromotion`'s "released" row cannot pass. Batten's unreleased refusal then sends the operator to a second release write.
- Fix: card 13 gives `promoted` (or `promoteTo` with recipe `raddle`) the draft-rule exemption itself, ahead of card 16, and asserts the empty status.

**MEDIUM — Card 22 (D8/D9), the two-returns halt names no step that can progress.**
- On a failure when `returns == raddleMaxReturns`, Raddle-File halts "for the parent, naming the failures and that the run used its two returns".
- A resume re-runs Raddle-File with `returns` still 2, so any remaining failure halts again.
- A `goto Raddle-Write` starts no new session (card 21 starts one only for a recorded, untaken return). The approval is not cleared by the halt, and `returns` is never reset, so Raddle-File's next failure halts again.
- No card names a way to re-propose, reset the count, or cancel the run. The halt needs a stated way forward, such as a goto that records a return-like reset, or `lym batten cancel`, plus a row reaching it.

**MEDIUM — Cards 19, 20, 21 (D7), "current Raddle-Write session" counts returns, not sessions.**
- `session` is raised only when a Raddle-File return is taken up (card 21). Card 19 says it is raised "at each entry that starts a new session".
- A Raddle-Write session that dies or times out and is resumed by `lym loom resume` is a new agent session that keeps the old number. Its `proposed_session` and `notice_session` still match, so:
  - `raddle-approve` needs no fresh `raddle-propose` in it;
  - no waiting notice goes out.
- That breaks D7's bound that "the waiting notice goes out in every session that approves". Cards 19 and 21 disagree, and no test row covers a resumed session after a died one.

**MEDIUM — Card 21, the interactive output file can predate a re-entry.**
- In interactive mode the seat's output is `approval.json`, and `Spec.validate` refuses an output file that already exists (`internal/shuttleengine/spec.go:28-35`).
- The approval is removed only on a taken return. A re-entry of Raddle-Write with an approval already written fails at spawn instead of progressing, for example a `goto Raddle-Write` after the halt above, or a resume after the session wrote `approval.json` but before the row moved on.
- No card names the reset, and no row tests it.

**LOW — Card 13, the `sketch`-label refusal names a step that writes nothing.**
- `validateLabels` refuses `sketch` on a non-raddle task, naming `--recipe raddle`.
- For an entry that is already a task, `promote` "returns a task unchanged, and nothing is written" (`boardcli` promote help). So the named step changes nothing.
- The way forward should be `lym board upsert` setting `recipe` to `raddle`, or dropping the label.

**LOW — Card 14, the promotion refusal names `lym board rename` with no target slug.**
- Card 14 has `newSlugError` name a slug both rules admit, but for a promotion it names only `lym board rename`.
- The note's rename is unjudged, so the operator can rename onto another slug the store holds and be refused again at promotion. The refusal should name an admitted target slug, as the other branch does.

**LOW — Card 17, `lym board raddle <unheld>` points at `lym board list`.**
- `lym board list` shows board entries only, not store slugs. After prune the Sketch and its filed slugs are gone from it, so the step cannot find a stored proposal.
- `lym report raddle` (card 26), which lists every stored Sketch, is the step that progresses.

**LOW — Card 24, the no-stored-record refusal names no command.**
- The refusal says "run its Discussion with a motif run first" but names no command.
- From the refused state (a task with recipe `raddle`, no record), progress needs `lym board upsert` setting `recipe` to `motif`, then a release and the run. After motif, the entry becomes a Sketch, and `lym board promote <slug> --recipe raddle` follows. The refusal should name those commands.

**LOW — Card 22, a failed `CreateBeam` has no stated verdict.**
- Card 22 names no verdict or way forward when the `lym beam create` child fails (only card 23 says "a failed run is returned").
- A returned error from `Call` versus `Stuck`/`Halt` decides whether resume re-runs step 4. The card should state `Stuck` with `Halt` and name `lym loom resume`, as BeamUp's steps do.

**LOW — Card 15, set-aside store files have no named next step.**
- `Apply` sets aside a store directory whose filed entry's add was refused or lost. Locally, `StoreDone` then reads step 3 as done from the local store, while origin holds neither the store nor the entries.
- No card says how the run or operator recovers: the Raddle-File re-run skips step 3, and nothing re-files.

**LOW — Card 21, a failed `AutoApprove` read has no verdict.**
- A failure of the `AutoApprove` key read at first entry has no stated verdict or way forward. It needs one, for example a returned error with "re-step the row".

Tool calls:
- Read `internal/boardengine/draft.go:1-70`, to confirm whether a note made a task is drafted by `applyDraftRule`; my context held only the subagent summary, not the rule's body.
