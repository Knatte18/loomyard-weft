```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism on one concrete case to check that every step gets the input the step before it produces.

**Findings**

1. **MEDIUM — D4: the resume rule has no input to read.**
   - Summary: "A resume keeps the passages its fresh launch wrote", but the resume path that runs Prepare again starts in a new run directory that does not hold the first launch's file.
   - Case: an agent is launched, `heddle.json` changes, then the agent is resumed through a spec with `ResumeSessionID`.
   - Evidence: `Runner.start`, reached from `StartSession`, makes a new run directory for every start, including a resume by spec (`internal/shuttleengine/run.go` around line 512). `Prepare(runDir, spec, …)` then writes `append-system-prompt.txt` into that new directory (`claudeengine.go:227-233`).
   - Nothing hands Prepare the earlier run's passages, so it can only re-read heddle. That breaks D4's purpose and its own acceptance case ("a resume after a heddle.json change keeps the first passages").
   - The other resume path, the `ResumeCmd` re-launch in the same directory, keeps the passages anyway because it does not run Prepare.
   - Fix: say where the first passages are kept so a resume by spec can find them, or drop the rule and accept a fresh read on resume.

2. **MEDIUM — D7 with D2: a combined Bolt commit can bury a heddle change in a set-aside tag.**
   - Summary: one commit can hold both a heddle edit and a board edit for a run that has ended, and today's set-aside rule would set the whole commit aside.
   - Case: an agent runs `lym heddle set x`, and a board status write for run `r` happens before the detached sync runs. `lym board sync` commits everything dirty in one "board sync" commit (`commitDirty` → `bolt.Commit`), so the commit touches `heddle.json` and `board.json`. Origin has meanwhile ended `r`.
   - In `replayBolt`, the commit touches only claimed paths. The run ledger names `r` as the only run it touches, and `r` is ended, so today's rule ("every run it touches is ended") sets the whole commit aside under a tag. The heddle entry change never reaches origin.
   - D7 only says "a commit no run-ledger path touches is never set aside". It does not cover a combined commit.
   - Fix: a commit that touches a claimed path outside the run ledger's files is never set aside whole. Either replay the non-ledger replayers' part or replay the whole commit by entry. Add an acceptance case for it.

3. **MEDIUM — D2: the write path described is not the board's write path.**
   - Summary: D2 says heddle writes go "pull first, write, commit, then the detached Bolt sync", "the same Bolt write path the board's writes use". Ordinary board writes do not do that.
   - Evidence: `Board`'s write path (`internal/boardengine/board.go` around lines 146-163) saves `board.json` under the write lock, renders, then calls `spawnSync`. It does not pull or commit. The detached `lym board sync` commits and pushes. Only the owner setter (`ownerset.go:106`) uses `Bolt.PullThenCommitWritten`.
   - Followed literally, a heddle write would pull and commit inline (in the foreground, needing the network), unlike every board write. That contradicts the stated rationale of matching the board.
   - Fix: describe the real path (save under the lock, then `boardengine.SpawnSync`), or justify pulling first on purpose.

4. **LOW — D6: `ffWorktree` also moves records branches.**
   - Summary: D6 fires only for the fast-forward of a code branch, but `ffWorktree` is shared and has no idea which kind of branch it is moving.
   - Evidence: `ffWorktree` (`originreconcilemoves.go:138`) has two callers (lines 197 and 392), one in the records or Bolt context.
   - The record should say the event fires at the code-branch call site, not inside `ffWorktree`. Otherwise "took in" fires for weft and Bolt moves with paths that are not warp paths. It renders nothing, but it is noise and a misread of the seam.

5. **LOW — D6 with D3: a freshly cloned hub may render nothing.**
   - Summary: `CloneHub` fires "made", and the render reads `heddle.json` from the Bolt checkout. If the warp worktree is wired before Bolt is cloned, the read finds no file and renders nothing.
   - The record never states the order, so the purpose (every worktree lym makes gets its files) can silently fail for the prime worktree.
   - Fix: state that "made" fires only after the Bolt checkout exists, or that `CloneHub` fires it last.

**Each decision, the case used, purpose met or not**

- **D1** (`set` a new `claude/prompt` entry with order 10, the same order as an existing entry): render order is by order, then slug, and an unknown key survives a rewrite through `internal/state`. Met.
- **D2** (`lym heddle set`, then the sync): the lock is right, but the stated path does not match the code (finding 3). Partly met.
- **D3, codex** (repo has `AGENTS.md`, one `codex/file` entry): the override is marker, then `AGENTS.md`, then the entry. Removing the entry later removes the marked override. Met.
- **D3, tracked file** (repo tracks `AGENTS.override.md`): no marker, so the target is skipped with a warning. Met; the gap is already under Open risks.
- **D4, fresh launch**: lym's sentences, a blank line, then the passages. Met.
- **D4, resume by spec**: not met (finding 1).
- **D5** (`Topology.Add` of a new pair): `/CLAUDE.local.md` and `/AGENTS.override.md` land once in the shared `info/exclude`, before the render. Pair removal leaves them. Met.
- **D6, Pull** (`Fabric.Pull` fast-forwards over a commit changing `AGENTS.md`): the paths diff from old to new HEAD and the override is re-rendered. The `ResetHard` branch also gives a correct diff. Met.
- **D6, reconcile and clone**: the call site is unclear (finding 4) and the clone order is unstated (finding 5).
- **D7, heddle-only commit on both sides**: the heddle replayer merges by entry with no conflict. Met.
- **D7, combined commit with an ended run**: not met (finding 2).
- **D8** (both sides change `text` of the same heddle entry): origin wins and the loss is reported. Met.
- **D9** (`lymcwd` exports the literal, `fabricengine` and `runproc` import `lymcwd`): no import cycle, since `lymcwd` imports only `dotgit` and `weftname`. The rename becomes a one-value change. Met.

**Tool calls**

1. Grepped `claudeengine.go` for the resume and run-directory handling, and `boardengine` for the sync spawn: to check D4's resume input and D2's write path.
2. Grepped `Fabric.Pull`'s warp arm for its moves and read `ffWorktree`: to check D6's call sites and their path input.
3. Read `board.go`'s write tail and grepped for `ResumeSessionID`: to confirm the board write path has no pull or commit, and how a resume is started.
4. Read `Runner.start`/`StartSession` in `run.go`: to confirm a resume by spec gets a new run directory.
