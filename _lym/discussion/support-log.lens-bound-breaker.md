```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I tried to break each bound, guard and claim about existing code in the record with a concrete counterexample.

**Findings**

1. **MEDIUM. D2: the record misdescribes the board's write path.** D2 says heddle uses "the same Bolt write path the board's writes use: pull first, write, commit, then the detached Bolt sync". The board has no pull-first step. It saves `board.json` and then starts `lym board sync` detached, and that sync commits whatever is dirty under `board.lock` and then pushes (header comment of `internal/boardengine/sync.go`). `Bolt.PullThenCommitWritten` is used only by `configcli/hubwide.go`, `fabriccli/fabric.go:745` and `anchormarker.go:191`. So D2 describes a path that does not exist. The plan writer has to choose between copying the board, which writes, then commits through the wildcard sync, and the pull-first `PullThenCommitWritten`, which commits each write's paths. The record should name one.

2. **MEDIUM. D7 and D2: one commit can carry a heddle change and an ended run's board change, and the set-aside rule then hides the heddle change.** Counterexample: the board's detached sync stages everything dirty in Bolt (`bolt.Commit("board sync")`). If heddle writes without committing right away, as the board does, a `heddle.json` edit and a board edit to run X can land in one "board sync" commit. Origin then ends X. `replayBolt` sets aside a commit when every changed run slug is ended, so the whole commit, heddle edit included, goes under `set-aside/X/<sha>` and the heddle edit is lost from the tip. The rule "A commit no run-ledger path touches is never set aside" says nothing about a mixed commit. D7 should say that only the ledger's paths are set aside and the other claimed paths still replay, or D2 should require that a heddle write is committed on its own under the lock.

3. **MEDIUM. D1: the guarantee that unknown keys survive a rewrite fails for an entry that `set` updates.** The bound in `internal/state/doc.go` reads: "A slice or array element merges with the old element at its index only when both decode to the same value, so an element updated in place, removed or moved loses its unknown keys." Heddle's entries are an array. `lym heddle set foo --order 5` therefore drops any unknown key a newer binary wrote on `foo`. Inserting or removing an entry shifts later indexes, so their neighbours can lose unknown keys too. D1's acceptance criterion "a store round-trip keeps … an unknown entry key" only holds for untouched entries. The board avoids this by keeping unknown keys on `storeFile` and `Task` itself. D1 should require that, or narrow the guarantee.

4. **MEDIUM. D4: "a resume keeps the passages its fresh launch wrote" covers one of two resume kinds and gives the other no mechanism.**
   - The in-place resume uses `Launch.ResumeCmd`, which is built at fresh launch and points at the same `append-system-prompt.txt`. It keeps the text with no change needed.
   - The adopt resume (`spec.ResumeSessionID`, claudeengine.go:146) runs a new `Prepare` with whatever `runDir` its caller passes, and rewrites the file. The record does not say where the first launch's passages come from when that `runDir` is new or the earlier file is gone.

   The acceptance criterion "a resume after a heddle.json change keeps the first passages" cannot be met as written for adopt.

5. **LOW. D3: the temporary file beside the target is visible in warp while it exists.** D3 writes "a temporary file beside it renamed into place" at the warp root. Only `/CLAUDE.local.md` and `/AGENTS.override.md` are excluded, so the temp file shows as untracked for that moment. A concurrent untracked or dirty check in the same worktree can see it, such as a remove dirty gate or a status run. The record should require a temp name that is also excluded, or one under `.lym`.

6. **LOW. D6: "fired wherever fabric moves a checked-out warp branch to origin's commits" names two `ffWorktree` sites but not which branches they move.** `ffWorktree` is called both at originreconcilemoves.go:197, in the generic per-branch move that also serves records and Bolt branches, and at :392 for a failed-push code side. Firing on every `ffWorktree` call would send weft and Bolt moves to heddle. The record should bound the seam to warp (code) branch moves explicitly.

**Claims attacked**

| Claim | Result |
|---|---|
| D9: `lymcwd` can hold the literal, and `runproc` can import it with no cycle (`dotgit` and `weftname` import nothing internal outside tests) | held |
| D9: `fabricengine` imports `runproc`, so the literal cannot live in `fabricengine` | held |
| D2: board write path is pull-first | broken (1) |
| D2: heddle shares the board's lock | held |
| D7: set-aside never affects non-ledger changes | broken for mixed commits (2) |
| D7: tags from a heddle loss collide with run tags | held; the tag name includes the SHA, and `SnapshotTagged` reads annotated tags only |
| D1: unknown entry keys survive a rewrite | broken for updated or shifted entries (3) |
| D4: resume keeps passages | partly held: in place yes, adopt broken (4) |
| D4: multi-line passages are safe in the append file | held; the one-line check covers lym's sentences only, and the file is not echoed |
| D3: a tracked `AGENTS.override.md` arriving by pull is handled | held; git overwrites the ignored file, the seam fires on that path, and the missing marker leads to a skip |
| D3: atomic write leaves warp clean | broken briefly (5) |
| D5: no unseed needed, one exclude file per repo | held |
| D6: a missing seam is a no-op and the root pre-run installs it outside tests | held |
| D6: the warp-only firing sites are clear | underspecified (6) |

**Tool calls**
1. Read the slice merge bound in `internal/state/doc.go`, which was not in context. Searched for the users of `PullThenCommitWritten` and `CommitWritten`, to check the board write-path claim. Grepped claudeengine for resume handling. Read the `ffWorktree` call sites at originreconcilemoves.go 185–200 and 385–395, to see which branch kinds they move.
2. Read claudeengine.go 140–178 for how resume sets up its run dir. Checked what `dotgit` and `weftname` import, for D9's no-cycle claim. Checked `Store.Save` for a sync or pull step, for D2.
