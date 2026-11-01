```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the diff (per function or branch: the case I ran, and whether it was correct).

**Per-function list**

- `replayBolt` / `replayClaimedCommit`, commit touching only `heddle.json` (no ledger path): `runs` is empty, `runsAllEnded` is false, so the commit goes by entry to `heddle` and is committed as one commit. Correct.
- Same functions, commit touching `board.json` and `heddle.json` where every ledger run has ended: `skipLedger` is true, so the ledger paths are dropped, the heddle part is applied and committed, and the original is tagged under `runs[0]`. Correct.
- Same functions, mixed commit where the heddle part ends up empty: `allEmpty` stays true and the aside is set, so nothing is committed and the original is only tagged. Correct.
- Same functions, a heddle loss: the tag is `replayer/heddle`. A ledger loss with an empty `Entry` falls back to `setAsideTag`'s `replayer/<ledger>`. Correct.
- `replayBolt` with no run ledger registered and a claimed commit present: `runLedger()` returns `ErrReplayerMissing` before the branch moves. A commit touching an unclaimed path is cherry-picked. Correct.
- `localLedgerRuns` / `readEndedSet` / `weftEndedByOrigin` / `readOriginOwners`: all read through `runLedger()`. With no ledger, `localLedgerRuns` fails only for a claimed commit. Correct.
- `heddleengine.Replayer.Apply`:
  - A field both sides changed keeps origin's value and is reported lost. Correct.
  - An added entry that origin also added reports a clash only for differing fields. Correct.
  - A removal of an entry origin changed is kept and reported lost. Correct.
  - An invalid merged or added entry is refused and the rest still lands. Correct.
  - `merged` starts as origin, so origin's unknown keys survive. Correct.
- `entrymerge.Change`, `AddClash`, `Same`: correct as far as I can run them. I did not compare against the pre-change board code.
- `Renderer.Render`: all cases correct.
  - A marked file for a target with no entry is removed, even when the target is off the exclude list.
  - An unmarked file is warned about and left alone.
  - A blocked `.heddle` writes nothing.
  - A stale `.tmp` is overwritten and then cleaned up.
  - `err` is correctly the `ReadFile` error inside the switch.
- `fileText`: the `@AGENTS.md` and `@.claude/AGENTS.md` imports appear only when there is no root or `.claude` `CLAUDE.md`. The `codex-file` target puts the root `AGENTS.md` first. Correct.
- `Seam.TookIn`: a take-in with no matching path takes no lock and renders nothing. Correct.
- `fireTookInOver` / `fireTookIn` / `fireTookInAt`, `pullWarpArm` and `reanchorAfterRewrite`: each sets `SeamWarning`, and `hubpull` appends it to the line detail. Correct.
- `originMover.move`: `headBefore` is read before `ffWorktree`, and the event fires only when `Moved`. The warning reaches `BranchOutcome.Warning` and `Summary`. Correct.
- `SeedWarpExclude`: passes a nil `rec`, and `Mutations.Append` is nil-safe (`mutation.go:157`). Correct and idempotent.
- `appendPromptText`: with no passages the text is byte-identical to before; otherwise each passage follows after a blank line. `StandingPassages` returns none for an empty `boltDir` and for a reader error. Correct.
- `Store.Set` and `ValidateEntry`: the flag names in the new-entry and no-target messages are wrong. See finding 1.

**Findings**

1. LOW, class `design`. The `heddle set` refusals name flags that do not exist.
   - Location: `internal/heddleengine/store.go:265-272` (`missingFlags` emits `--target`) and `store.go:297` (`ValidateEntry` says "give at least one --target"). The real flag is `--targets` (`internal/heddlecli/verbs.go:146`).
   - Case: `lym heddle set x --text-file f --order 1` on a new entry gives "needs --target", a flag name the CLI rejects.
   - Case: `--targets ""` makes `splitTargets` return nil. On an update that silently leaves the targets unchanged instead of refusing, and on a new entry it hits the same wrong message.
   - D2 requires each refusal to name a way forward that works.

2. LOW, class `scope`. `FastForwardPair` fires a take-in event that D6 does not list.
   - Location: `internal/fabricengine/parenttips.go:284`.
   - D6 limits take-in to the `Fabric.Pull` warp arm and `ReconcileOrigin`'s code-branch fast-forward, and says a parent merge or landing renders nothing. The plan's card 10 does name the parent-pair fast-forward, so the plan licenses it, but the decision record does not.
   - The returned warning text is also discarded. It is logged only, so it never reaches a result. Whether this is a defect depends on whether that fast-forward is a take-in from origin.

3. LOW, class `design`. Every Claude launch can stall for up to 30 seconds on the Bolt write lock.
   - Location: `internal/heddleengine/render.go:146-160` (`read`, used by `Passages`), called from `claudeengine.Prepare`.
   - A Bolt push holds that lock across its fetch and push retries. If the wait runs out, the launch drops all `claude-prompt` passages and only logs a warning.
   - D4 says an unreadable store gives lym's sentences only. A silent loss of standing rules after a 30-second stall is a tolerable but notable path.

4. LOW, class `consistency`. `Decode` bakes a placeholder path into its way-forward text.
   - Location: `internal/heddleengine/store.go:162`, used by `replayer.go:63-73`.
   - A replay of a malformed historical `heddle.json` blob tells the operator to run `git -C <Bolt checkout> checkout HEAD -- heddle.json`. That advice would restore the file from HEAD, which is not the blob that failed to decode.

No BLOCKING or MEDIUM correctness defect found in the replay, merge, render, seam or prompt paths.

**Tool calls**
1. Read `target.go`, `passages.go` and the diffs of `pull.go`, `claudeengine.go`, `settings.go` and `parenttips.go`, plus `originreconcilemoves.go:385-430`. These were not in my context, because the manifest only had the diff for main, `add`, `clone` and `hubpull`.
2. `grep` of `Mutations.Append` nil-safety and `mutateGitExclude` in `fabricengine`, to check that `SeedWarpExclude`'s nil `rec` is safe.
