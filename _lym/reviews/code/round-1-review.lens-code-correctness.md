```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I ran each function and branch the diff adds or changes on concrete inputs.

**Functions and branches run**

- `gatefixer.LastTurnMessage` (strand.go:27-48):
  - missing file: `ReadFile` gives ENOENT, so `""` and nil. Correct. An empty `EventsPath` takes the same branch.
  - offset past EOF: clamped, parses empty bytes. Correct.
  - newest stop whose message is only whitespace: trims to `""`, so the batch stays dead/asking. Correct.
  - `ParseEvents` error: propagated. Correct here; see finding 3 for what the caller does with it.
- `awaitTerminal`, asking branch (recoverbatch.go:870-880):
  - dead/asking with a message: becomes a waiting digest. `HeadSHA` is empty, so the head reconcile is skipped, the strand is stopped and the run dir is kept. Correct mechanically.
  - dead/asking with no message: stays dead. Correct.
  - dead from timeout or death: untouched. Correct.
  - the question reaching the strand that resumes the batch: see finding 1.
  - turn ends that are not questions: see finding 2.
  - `LastTurnMessage` error: see finding 3.
- `askingNeed` (waiting.go:16): `"  a\n b  "` becomes `"the recovery stopped to ask: a b"`. Correct.
- `PersistRecoveryTerminal` with the waiting digest from asking: `recordWaiting` archives a report that is absent, so nothing happens there. The batch stays non-terminal and waiting, with its count unchanged. Correct.
- webstercli terminal switch, waiting: exits Ok with `recovery_retry` false. Correct.
- `RecoverSpawnOrAttach` / `recoverSpawn` on a waiting prior: not attached, the cap is skipped, the spawn is uncounted and the prior strand is stopped. Correct, except that the prompt carries nothing of the question (finding 1).
- `recoveryRetryRead`:
  - dead recovery, Recoveries 2, unrendered amendment: true, 2.
  - same with only a rendered amendment: false.
  - Kind fork with an amendment: false.
  - non-terminal: false.
  - Correct, and the amendment check comes before the git read, so no read failure can mask it.
- `Rebaseline` restamp loop (rebaseline.go:361-380):
  - terminal dead batch, card named: restamped, recorded in `AmendedCards` as unrendered, kept out of `CardsAmended`.
  - an existing rendered entry is reset to unrendered.
  - in-flight batch: listed in `CardsAmended`.
  - Correct.
- Re-arm end to end: dead at 2 recoveries plus an amendment.
  - `deadEndBatchError` skips the batch.
  - begin-batch's `existingReportRemedy` names recover-batch once more.
  - the cap refusal is skipped.
  - the spawn is uncounted and marks the amendment rendered.
  - a second death gives no retry, so each card edit buys exactly one recovery.
  - The same holds for terminal failed and stuck batches.
  - Correct.
- `runplan.RecordedOverviewFrame`:
  - frame hash present: returned.
  - no overview hash: `CauseNoHash`.
  - copy absent (ENOENT): `CauseBaselineAbsent` plus the path.
  - copy without a Card Index: `CauseNoCardIndex`.
  - EACCES on the copy: error returned.
  - Correct. A nil `st` is guarded by every caller.
- `overviewFrameEditedFindings` / `unframedOverviewFindings`:
  - nil `st`, or no batches: none.
  - frame present: compared.
  - no hash: none, which matches the rebaseline's `len(PlanFileHashes) &gt; 0` guard.
  - copy absent and overview changed: blocking finding, which matches the rebaseline's refusal (`overviewIndexOnly("")`).
  - copy absent and overview unchanged: none.
  - read error: returned.
  - Correct.
- `ValidatePlan` run dir: `RunDirs(impl)` gives `websterengine.Dir` for webster. Weaver records its baseline copy through `RestampBaseline` into its own run dir. Correct.
- `createdHandleGlyphs`:
  - Create refs are `plan:` handles (parse.go:781), so `HandleBody` strips the prefix.
  - a forthcoming card maps to 0; the earliest current card wins.
  - Correct.
- `createdBy`: quarry's not_found member result carries `ID: g.String()` (engine resolve.go:324), so a canonical handle body matches a target spelled another way. Correct.
- `statusFindings` / `missingDetail`:
  - creator 3, using cards 2 and 4: card 2 gets the "created by card 3" detail, card 4 nothing.
  - the creating card itself: passes.
  - forthcoming creator (0): every card passes.
  - unit not found: says the unit does not exist.
  - found unit with an empty member list: no list suffix.
  - Correct.
- `missingMemberSiblings`:
  - skips found results, results whose unit is not found, and created targets.
  - a `ParseRef` error on a quarry-resolved target cannot happen in practice.
  - Correct.
- `siblingMembers`:
  - owner missing: Expand returns not_found with a nil error (engine expand.go:247-252), so the list is empty and there is no hard error.
  - owner a func: `NotATypeError`, so nil.
  - owner ambiguous: no members.
  - owner-less `sub_test` unit: the first `Glyphs` call fails and the fallback reads `sub`. The Unit filter keeps the external-test symbols only.
  - Correct: no ordinary glyph-not-found becomes a hard error.
- `fingerprintMismatchWayForward` moved onto `runplan.RecordedOverviewFrame`: same semantics, only the error prefix changed. Correct.
- `GoPackageDir` export: a rename only. Correct.

**Findings**

1. **BLOCKING**, design. `internal/websterengine/recoverbatch.go:317,385,892`.
   - A recovery resumed after an asking wait gets neither its question nor the parent's answer.
   - `awaitTerminal` stops the asking strand (line 892).
   - The resume spawns a fresh strand uncounted (line 385). Its prompt takes `failureDigestBlock(prior)`, which renders only failed reasons and amendments (lines 168-182). No input carries the need or the answer (line 317).
   - So the answer goes nowhere, and a fresh strand that hits the same uncertainty asks again. Nothing bounds that loop of uncounted waits.
2. **MEDIUM**, design. `internal/websterengine/recoverbatch.go:872-880`.
   - Every dead/asking turn end that carries any message becomes waiting, whether or not it is a question.
   - A Claude Stop hook almost always carries `last_assistant_message`. A recovery that ends its turn on a summary such as "all cards committed" without writing a report now waits on the parent with that summary as its need.
   - Before the change it was dead/asking under the commit-and-count retry rule. Combined with finding 1, it loops uncounted.
3. **LOW**, design. `internal/websterengine/recoverbatch.go:873-876` against `:890-894` and the doc at `:805`.
   - A `LastTurnMessage` error returns before `StopStrand`. That breaks the documented rule that every terminal digest removes the recovery strand.
   - The live strand and the non-terminal record stay behind, and the next call attaches and fails the same way.
   - For Claude, `TurnEndedAfter` may decide through session signals rather than `ParseEvents`, so the earlier poll succeeding does not rule this out.

Tool calls:
1. `grep` of `HandleBody`, the Create grammar and quarry's not_found result shape: my context did not show how Create refs are spelled or what ID a not_found answer carries.
2. `grep`/`sed` of planparser's group-ref parsing and quarry's `ResolveResult` constructors: to confirm Create refs are handles and that a not_found member result carries `g.String()` as its ID.
