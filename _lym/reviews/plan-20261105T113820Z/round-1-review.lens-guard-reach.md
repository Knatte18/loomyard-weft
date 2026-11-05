```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, refusal, branch or test row the cards add, I traced whether a real entry point can reach it.

**Per guard: reachable or not**

- **Card 13**
  - `checkDiscussedPromotion`: reachable through a plain promote, or a promote with `--recipe darn`, of a discussed note stripped of its `sketch` label, and through an upsert of such a note to a task.
  - `validateLabels` refusing `sketch` on a non-raddle task: reachable through a plain promote of a Sketch. It is also reached on a state the card never names (finding F4).
  - The released list in `boardCriticalSection`: reachable.
  - `checkDiscussedPromotion` recurs unguarded in `FabricReplay` (finding F5).
- **Card 14**
  - Store-slug refusals: reachable through an upsert under a filed slug; through a rename to a held Sketch slug once the Sketch is pruned; and through `promote --recipe raddle` of a filed part that a motif run's end made a note again.
  - The `created` exemption: reachable only through a contrived sequence (task made a note, then promoted again), but D10 mandates it.
  - Store read error: reachable through a corrupt `store.json`.
  - The guarded site recurs unguarded in the replayer for any recipe but raddle (finding F1).
- **Card 15**: set-aside on a lost or refused filed add is reachable through a cross-machine replay.
- **Card 16**
  - `FileRaddle` done marker: reachable on a retry.
  - `ErrRaddleRecheck`: reachable when the board moves between the pre-check and the write.
  - Dropping a `depends_on` on a gone entry: reachable.
  - `ReleaseTasks` and `EndRaddleEntry` skips: reachable after a prune or a retry.
  - The type-label refusal `validateLabels` puts on a filed project: reached on every `project` and `beam` filing, which the card doesn't plan for (finding F3).
  - `checkTaskCreation` is not reached: `OpenHub` sets no `StrandName`.
- **Card 18**: the `ReadStored` found and not-found branches are reachable.
- **Card 20**
  - The no-`RaddleWrite`-row refusal is reachable from a darn worktree.
  - The wrong-row refusal is reachable on a raddle run halted at Raddle-File or Record-Write.
  - The approve refusal before a propose in this session is reachable after a return or any re-call of the row.
  - Invalid-proposal and moved-board refusals: reachable.
- **Card 21**
  - The `AutoApprove` error is reachable through a config error.
  - The fresh-entry clearing is reachable after a return or a goto, and its "already taken up" branch on a crash-resume.
- **Card 22**
  - The `StoreDone` skip, the hash mismatch, the return, the halt at the return cap, and the `CreateBeam`, `Release`, `Notify` and `EndEntry` halts are all reachable.
  - A step-2 `SplicePart` error is reachable from a proposal that passes D6, with no prescribed handling (finding F2).
- **Card 24**
  - The no-record refusal is reachable through a hand-upserted raddle task that is then released.
  - Store-held as another Sketch's, under another `created`, or as a filed slug: reachable only through the replay bypass, since promotion meets it first.
  - The own-stamp pass branch is reachable, but not by the state its rationale names (finding F6).
- **Card 25**
  - The marker-absent and marker-present branches, and the `HandOff` and `Notify` error halts, are reachable.
  - A post that reaches no one goes unseen (finding F7).
- **Card 12**
  - `slug-sketch` is shadowed by `slug-taken`, so it never fires alone (finding F8).
  - The other checks are reachable.
- **Card 3**: the `stopRecorded` hook is reachable through `samplersession.go:247` `StopStrand` and the tackler stop. The handle-form `Run.Stop` with no `Wait` is not hooked; I found no path that reaches it unawaited.
- **Cards 6, 7, 8, 9, 11 and 17**: their branches and refusals are reachable (`DiffSize` skip on a resume past the merge, `cost_min_runs` under 1, a run with no `run.json`, an invalid `raddle` key, an unheld slug).

**Findings**

- **F1 — MEDIUM, design, card 14 (Bound) and card 24.**
  - Card 14 says batten's start guard (card 24) meets an entry that the replay landed under a held slug. Card 24's `hubStartGuard` judges only entries whose recipe is `raddle`.
  - So a darn or tapestry task, or a project, replayed under a filed or Sketch slug is met by no guard. `lym board raddle &lt;slug&gt;` then maps it to the old proposal, which breaks D10's "a slug in the store always names the entry that filed or was raddled under it".
  - Evidence: `internal/boardengine/replay.go:465`/`:528` add through `validateWrite` alone; `internal/battencli/startguard.go:115-206` has no slug-store check; card 24 Intent: "when the entry's recipe (`boardRecipe`) is `raddle`".
- **F2 — MEDIUM, design, cards 12 and 22.**
  - D6's checks (card 12) have no "a part builds a decision id the record lacks" check, and no "a part carries a PATTERN id the record lacks" check. `decision-unbuilt` only covers record ids no part builds, and `criterion-unknown`/`risk-unknown` cover bullets only.
  - Such a proposal passes the gate, `raddle-approve` and Raddle-File's check. It then reaches `SplicePart`'s unknown-id error at step 2 (card 11).
  - Card 22 prescribes return handling only for check, hash, re-check and `File` errors. A step-2 splice error comes back as a bare returned error with no return to Raddle-Write and no way forward.
  - Evidence: card 12, the Check list; card 11, "A listed id the record does not hold … is an error naming it"; card 22, the Call steps.
- **F3 — MEDIUM, design, card 16 (`FileRaddle` project).**
  - `validateLabels` refuses a project with no type label (`labels.go:39-40`). Card 16's project entry is filed with "no `sketch` label", and neither card 16 nor card 10's spec (`project: {slug, title}`) gives it a type label.
  - So every `project` and `beam` filing hits this refusal, and the card names neither the refusal nor the label's source.
  - Evidence: `internal/boardengine/labels.go:39`, card 10 Keys, card 16 `RaddleEntry`.
- **F4 — LOW, design, card 13 (`validateLabels`).**
  - Today a plain promote keeps a Sketch's `sketch` label on the task (`store.go:771-774` copies labels). Tasks carrying `sketch` with a non-raddle recipe can therefore already exist on the board.
  - After card 13, every write that runs `validateWrite` on such a task is refused: upsert at `store.go:568`/`643`/`661`, and replay at `replay.go:465`/`528`. No card names this state, a migration, or a test row for it.
- **F5 — LOW, design, card 13 (`checkDiscussedPromotion`).**
  - The rule runs only in `boardCriticalSection`. A note-to-task promotion of a discussed note from another machine's board commit replays through `validateWrite` alone (`replay.go:465`/`528`).
  - Once the note's label is stripped, a non-raddle promotion lands on origin unjudged. Card 13 states no bound for this, as card 14 does for its own rule.
- **F6 — LOW, consistency, card 24.**
  - The pass branch "its own Sketch slug with its own `created` … so a resumed raddle run past its filing starts" can't be reached by a resume. `refuse` runs the check only "for a run or step with no status file", and a resumed run has a status file (`startguard.go:168`, `:195`).
  - The branch is still needed, for a re-run after a cancel without `--remove`, but the stated reaching state is wrong.
- **F7 — LOW, design, card 25.**
  - `pawlengine.Deps.Notify` is `func(line string) error` (`internal/pawlengine/deps.go:51`). It drops `NotifyOrchAt`'s `posted` result, which is false with nil error when no orch strand is recorded (`orchcli/notify.go:68`).
  - So a hand-off that posts nothing still writes the marker, and the hand-off is lost for good. The card doesn't guard or bound this state.
- **F8 — LOW, consistency, card 12 (`slug-sketch`).**
  - During the run, the Sketch is the run's own entry, open on the board until step 7. A part slug equal to it always also raises `slug-taken`.
  - So `slug-sketch` never fires alone, and TestCheck's row, "breaking that rule alone and asserting exactly its finding", can't hold unless `slug-taken` exempts the run's entry. The card doesn't say which.

**Tool calls**

1. One Bash call covering four lookups the manifest lacked:
   - `validateWrite`'s call sites, to see whether every task or only the incoming one is validated, for F4 and F5.
   - `StopStrand`'s callers, to trace card 3's stop-path reach.
   - `pawlengine.Deps.Notify`'s signature, for F7.
   - `replay.go`'s add path, to see which rules a replayed entry meets, for F1 and F5.
