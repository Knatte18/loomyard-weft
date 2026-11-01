```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked every pair of decisions in the warp-visibility record that act on the same state, then ran the acceptance criteria and constraints against the decisions they touch.

**Findings**

1. **MEDIUM, D7 × D2/D1: a commit touching both the board and heddle can be set aside whole, so the heddle change is lost to a tag.**
   - D7 says "a commit touching only claimed paths goes to the replayer of each path it touches" and "a commit no run-ledger path touches is never set aside". It says nothing about a commit that touches both `heddle.json` and the board entry of an ended run.
   - Today's rule (`replayBolt`) sets aside a board-only commit when every slug it changes belongs to an ended run. Carried over unchanged, a mixed commit whose board slugs are all ended goes under a tag, and its `heddle.json` change goes with it.
   - Mixed commits can happen. `Bolt.Commit` "stages and commits every change in the Bolt's repo", and the board's detached `Sync` (`commitDirty` → `bolt.Commit`) commits whatever is dirty, a `heddle.json` left over by an interrupted heddle write included. The by-entry case in D7's acceptance criterion presumes that.
   - Fix: D7 should say a commit is set aside only when it touches no claimed path outside the run ledger's. Otherwise only its run-ledger part is dropped and the other replayers still apply theirs.

2. **LOW, D2/D9 × D7's acceptance criterion: `board.lock` is a board file name that stays in fabric.**
   - D7's criterion says "no board file name … is left in `internal/fabricengine`".
   - D2 keeps the Bolt write lock as `fabricengine.BoardWriteLockFile`, and D9 keeps the file name and moves only its directory to `BoltDirName`.
   - Fix: narrow the criterion to the replay seam's file names (`board.json`, overview and design files), or rename the constant to Bolt's.

3. **LOW, D1 × D8/D7: no decision says heddle's replay write keeps unknown keys.**
   - D1 requires every rewrite of `heddle.json` to go through `internal/state` and keep unknown keys.
   - D8 and D7 have heddle's replayer write `heddle.json` during a replay, and D1's criterion tests only the store round-trip.
   - Fix: state that the replay write uses the same tolerant rewrite, and add a replay case keeping an unknown key to D8's criterion.

4. **LOW, D6 × D5 × D3: the order of exclude seeding and the "made" render is only implied.**
   - D3 refuses to render a target whose path is not on the exclude list.
   - D6 fires "made" "once the worktree is wired", and D5 seeds the exclude lines on creation and re-wire.
   - The record never says the exclude lines are written before "made" fires. If seeding came after, the files would show as untracked in that window. The refusal itself reads config, not the exclude file, so it is not affected.
   - Fix: one clause in D6 saying "made" fires after D5's seeding.

5. **LOW, D7 × D8 (refusal) × the set-aside tags: a refused heddle entry is tagged under the run-slug namespace.**
   - Today a refusal sets the commit aside under `set-aside/<slug>/<sha>`, with the slug taken from the finding's entry.
   - For heddle, that entry is a heddle slug, which could match a run slug, so a tag for a heddle refusal would read as that run's.
   - Fix: D7 should name what tag a non-ledger replayer's set-aside carries.

**Pairs checked**
- D2 × D7 (a heddle write pulls first and replays local heddle commits): held, since the root pre-run registers heddle's replayer.
- D2 × D9 (the write lock's path comes from `BoltDirName`): held.
- D2 × D6 (a `heddle.json` write does not reach live worktrees): held.
- D1 × D8 (unknown keys on a replay rewrite): not stated, finding 3.
- D1 × D3 (order and ties used when rendering): held.
- D1 × D4 (order of prompt passages): held.
- D3 × D5 (a target must be on the exclude list): held, since the template default lists both files.
- D3 × D6, "made": render order relative to seeding is only implied, finding 4.
- D3 × D6, "took in": held, including a tracked `AGENTS.override.md` or `CLAUDE.local.md` arriving (the file has no marker, so the render is skipped with a warning).
- D3 × D6, reset-hard in `Fabric.Pull`: held, since ignored files survive the reset.
- D3 × D3 (concurrent renders): held, by the atomic rename.
- D5 × D3 (never unseeding against removing a stale render): held.
- D5 × D9: no shared state.
- D4 × D9 (the seam keyed on the Bolt dir, the `Board` field of the dirs): held.
- D4, resume against fresh launch: held as stated. It needs the passages stored apart from the file `Prepare` rewrites, which the plan can work out.
- D6 × D7 (a Bolt pull is not a warp take-in): held.
- D7 × D8 (refusing invalid entries): held. The tag namespace is finding 5.
- D7 × D9 (run ledger against `BoardDirName`): held.
- D7, mixed commit with an ended run: broken, finding 1.

**Acceptance criteria and constraints against the decisions**
- D7's criterion against D2 and D9: conflict, finding 2.
- D8's criterion against D1: incomplete, finding 3.
- D5's criterion "a removed pair leaves the lines" against D5's never-unseed: held.
- D6's criterion against D3's refusal: held.
- The PATTERN-fabric-git constraint against D6 (fabric computes the changed paths, heddle runs no git): held.
- The PATTERN-cwd-resolution constraint against D2's `render [<worktree>]`: held, through `cliwire`.
- The PATTERN-tolerant-records constraint against D7 and D8: finding 3.

**Tool calls**
- Read the `Bolt.Commit`, `CommitWritten` and `PullThenCommitWritten` docs in `internal/fabricengine/bolt.go` to confirm whether Bolt commits stage every dirty file. My context held only their signatures, and finding 1 depends on mixed commits being possible.
