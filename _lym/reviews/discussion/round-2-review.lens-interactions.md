```yaml
lens: interactions
outcome: reported
```

Task: run the interactions lens over this record's decisions. Five pairs break; every other pair holds.

**Pairs checked**
- D4 × D5 (load refusal vs. reconcile move, for local, worktree and hub base files): held. The move runs before the prune, and each layer's way-forward command runs the move.
- D4 × D6 (profile-block refusal vs. profiles.yaml move): **broken for a local profiles file** (F-int-1).
- D4 × D7 (Seed-Child refusal vs. board carry): held. Seed-Child's wrapper prints the one way forward while the carry waits.
- D4 × D9 (exact-path old-name check vs. new-path categories): held. The criteria on `discussion: {reviewer_fan: ""}` and `discussion: {timeout_min: 60}` both agree with D4's shape rule.
- D4 `--unset` × D4 check: held, because `configengine.Unset` validates nothing (`internal/configengine/set.go:114-134`).
- D5 × D6 (which files configsync moves): **broken**. D5 says "every file it reconciles", and a profiles local file is reconciled but never moved (F-int-1).
- D6 × D7 (the shared configreg helper): **broken**. A helper over a module-keyed object cannot keep YAML comments (F-int-2).
- D6 × D5 collision rule × FollowTemplate: held for these orders:
  - old-build re-add, then new-build walk: the operator's value at the new path wins.
  - move of file and base, then follow: the operator's change shows up as `existing ≠ base` and is kept.
- D7 × walk stamp (`ensure`) and the beam-settings carry: held. A skipped walk leaves the stamp stale, and `carryBeamTaskSettings` also waits for the retry (`internal/hubreconcile/walk.go:81-84,162-164`).
- D7 × the hub-wide write skip: held.
- D5 × D8 (`FlagCodeAutoAccept` reads the moved key): held. It reads only the hub base file, which configsync moves before reinterpretations run (`internal/loomengine/reinterpret.go:27-47`, `internal/hubreconcile/walk.go:228-243`).
- D8 merge-parent nesting × D4/D7: held.
- D1 × D2 × D3 (drift stays flat; full names): held.
- D5 bound (machine layer untouched) × D4: held.
- Constraint "configsync before reinterpretations" × D8: held (board files only, which is all the reinterpretation reads).
- D5 criterion "the walk mails the orch" × D5/D6/D7 mail lines: **broken** (F-int-3).
- D6 "untouched file parses to the new template" criterion × D6: held. The moved base equals the moved file, so the follow takes the new template.
- D7 criteria × D7: held.

**Findings**

F-int-1 — MEDIUM, design, `_lym/discussion/decision-record.md:125` (D6), with D5 at `:108`
Summary: D6 moves only the hub `profiles.yaml`. A profiles local file is reconciled but left untouched, so its old loom and tapestry keys are never moved. Seed-Child's D4 refusal then names `lym config profiles` from the prime, which edits the hub file, not the file that holds the old key.
- That breaks the Goal's "every layer" carry-over.
- It also breaks D5's "every file it reconciles" claim.
Evidence:
- `internal/runprofile/runprofile.go:54-55,162`: profiles is HubWide with WholeEntries, and `Load` reads `StackLayers`, so a local layer applies whenever Anchor is set (`internal/configengine/layers.go:143-146`).
- `internal/configsync/configsync.go:235` and `:310,323`: `ReconcileAll` reconciles every module's local file, and a seed-only present file is untouched.
- `internal/battenshed/seamchild.go:155`: the wrapper names only `lym config profiles`.
Fix: D6 also moves a profiles local file's blocks whenever `ReconcileAll` reconciles it. Otherwise, state that profiles has no local layer, or name a local-file way forward in D4.

F-int-2 — MEDIUM, consistency, `_lym/discussion/decision-record.md:129-130` (D6), with D7 at `:139`
Summary: D6 says it moves through the D7 helper and keeps the file's comments. D7 defines that helper as applying move tables to a module-keyed settings object, the board's decoded JSON map. A helper over a decoded map cannot keep the YAML comments of `profiles.yaml` or its template base, so the two decisions contradict each other.
Evidence: `internal/boardengine/task.go:42-44` (RunSettings is `map[string]any`) and `internal/yamlengine/follow.go:53-56` (the profiles file is node-based and keeps its comments).
Fix: Say the helper works on a YAML node tree that D7 encodes from the board's object. Otherwise, give D6 its own node-level move that shares only the move tables.

F-int-3 — LOW, consistency, `_lym/discussion/decision-record.md:116` (D5), against the D5 criterion at `:217`
Summary: D5 says the walk mails one line per walk naming each moved file. Worktree and local moves happen in `reconcileWorktree`, which only logs its results, once per pair, after the hub-wide notices are already sent. The record does not say where these lines are gathered and mailed, so one walk can't produce the single line the criterion pins unless the walk gathers them.
Evidence: `internal/hubreconcile/walk.go:271-272` (notices are sent inside `reconcileHubWide`) and `:346-351` (`reconcileWorktree` only calls `logResult`).
Fix: State that the walk gathers the move lines from the hub-wide pass and from every worktree, and mails them once at the end of `walkHub`.

Tool calls:
1. Grepped `FlagCodeAutoAccept` in `internal/loomengine/reinterpret.go`, to check whether D8's reinterpretation reads worktree files the walk moves later; the manifest did not include that file.
2. Grepped `runprofile.ConfigSpec` and `Load` in `internal/runprofile/runprofile.go`, to check whether profiles has a local layer that D5 and D6 leave unmoved; the manifest did not include that file.
