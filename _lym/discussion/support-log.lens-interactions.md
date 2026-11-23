```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked pairs of decisions that act on the same state, and the acceptance criteria and constraints against the decisions they touch.

**Findings**

1. **MEDIUM — D4 and D5 (D4's derived refusal).** A refusal derived from the retired-key mechanism does not refuse most moved keys at the local layer.
   - D4 derives the refusal from `MovedKeys` through the retired-key path. Under that path, `configengine.ValidateLocal` judges a unit a retired key covers by that key's category alone, and "CategoryNone admits it with no shape check".
   - Most moved loom keys are `none`, for example `discussion_review_ask_parent` and `code_review_ask_parent` in loom's `ConfigSpec`. If a derived retired entry kept the old key's category, a local file or run_settings holding `code_max_bounces` would pass validation. The loader would then ignore it, so D4's promise is broken and the value is lost silently.
   - The record must say that a moved key is refused whatever its category, by a path separate from the category-judged retired path. Otherwise, AC "D4: an old key in … a local file … is each refused" cannot pass with the mechanism D4 names.

2. **MEDIUM — D4, Constraints and PATTERN changes (base-layer load).** D4 makes the loader refuse an old key in a worktree file or hub base file. Today a key the template lacks in a base layer only warns.
   - `PATTERN-config-layers`'s "Keys the template lacks" says such a key, retired ones included, "the load warns of it and no loader reads it".
   - The listed `PATTERN-config-layers` change covers only "a key move a move table". The plan-writer note covers only the reconcile side. Nothing records that strict loads now refuse a moved key in a base or worktree file.
   - An unrecorded departure from a PATTERN entry is a finding. The fix is to state it in D4 and in the background note.

3. **LOW — D7 and the walk's hub-wide Bolt write (failure and restart).** D7 writes `board.json` through `boardengine`'s own write path and lock. D7 does not say whether that write runs inside `reconcileHubWide`'s `BoltWrite`, which snapshots and restores only config files, or as a separate step.
   - Run inside: a nested board lock and commit within `PullThenCommitWritten`. A later failure restores config but leaves `board.json` moved and removes the marker.
   - That outcome is harmless only because the move is idempotent, and the record does not say so.
   - The fix is to state where D7 runs and that the board is written before its marker commits. Idempotence alone keeps a re-run safe.

4. **LOW — D5, the open risk on mixed builds, and the local layer.** An older build's reconcile on another machine prunes every key its template lacks from local files ("A local override file is pruned of every such key on any reconcile").
   - It would therefore strip the new nested keys that D5 or Seed-Child wrote to that machine's local files, not only refuse to load the hub base.
   - The open risk names only the refusal. It should also name the silent pruning of local files.

5. **LOW — D4 and `lym config --unset`.** D4 refuses `--unset` of an old name. `PATTERN-config-layers` names `lym config <module> --unset <key>` as the operator's way to remove a key the template lacks.
   - D5 normally moves the key away first, so the refusal only bites on a file the walk skipped, such as an incomplete pair.
   - There the operator loses the documented removal route. Either bound the refusal to `--set`, or say that `--unset` of an old name removes it.

**Pairs checked**
- D1 × D5 (bare `discussion`/`plan`/`parts` scalar against the new map, told apart by shape): held.
- D1 × D4 (loom's retired `pr_review` against the new `pr` area; `retiredCovering` matches `key+"."`, so there is no overlap): held.
- D1 × D9 (open maps and categories renamed to paths): held.
- D4 × D5 (a refused old name against the reconcile that moves it; `Ensure` runs before any command): held for walked files. The category gap is finding 1.
- D4 × D5 (an incomplete pair skipped by the walk keeps old keys and is then refused): held only through the refusal's way forward; see finding 5.
- D4 × base-layer warn rule: broken (finding 2).
- D4 × D6 (`checkProfiles` refuses flat profile blocks): held. `configsync.ReconcileHubWideAt` runs no CheckLoad, and D6 runs in the same write before any validation. A D6 failure fails the whole write and the stamp retries.
- D5 × D6 (profiles is seed-only and carries no move table, so only D6 touches it): held.
- D5 × D8 (`FlagCodeAutoAccept` reads `code.auto_accept` after configsync moved it; walk order confirmed): held.
- D5 × D7 (Seed-Child between the moves: both run in `Ensure` before commands): held.
- D6 × configsync's three-way merge (untouched profiles take the new template; tuned ones are moved by D6): held.
- D7 × D4 (moved settings pass `PrepareRunSettings` at the board write and at Seed-Child): held.
- D7 × hub-wide BoltWrite restore: held only through idempotence (finding 3).
- D3 × D4 (refusals render names through the helper): held.
- D2 × D3 (drift flat, named `drift.judge_ask_parent`): held.
- AC D5 "local file keeps values through a prune" × D4's derived retired keys: held, since the move precedes the prune.
- AC D4 "worktree file refused" × D5: held, provided the test writes the old key after reconcile.
- Constraint "walk runs configsync before reinterpretations" × D6/D8: held (confirmed in `walk.go`).
- Open risk mixed builds × D5: incomplete (finding 4).

**Tool calls**
- I read `internal/hubreconcile/walk.go` lines 199–275 and grepped `internal/configsync` for CheckLoad and validation. My context held only the line numbers of the hub-wide write, not its snapshot/restore scope, and I needed to know whether configsync validates profiles before D6 runs.
