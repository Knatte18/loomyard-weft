# Card 3 — hubgeom-autonomy-reader

**Create:**
- `internal/hubgeom#Autonomy`
- `internal/hubgeom_test#TestAutonomy`

**Extends:**
- `internal/hubgeom/doc.go`

**Uses:**
- `internal/hubgeom#TaskSettingChain`
- `internal/dobbyengine#LookupAutonomy`
- `internal/hubgeom_test#TestTaskSettingChain`

**Intent:** D2 and D11: the one reader every autonomy consumer calls, at call time, for a run or for an orch.

`Autonomy` answers the autonomy of the worktree `slug`, looked up through `hubgeom.TaskSettingChain` and `dobbyengine.LookupAutonomy`.
`location` is any worktree of the hub and only locates it.

- A run's reader passes the run's own pair slug, so the lookup starts at the run's own worktree.
- An orch's reader passes its own worktree's name, so the lookup starts at its own worktree, never its parent's.
  The prime's own name reads the prime's chain: `slug` equal to the prime's worktree name is passed to `TaskSettingChain` as `""`.
- The prime is resolved spawn-free from the hub's git files: the main worktree `dotgit.ListWorktrees(location.WorktreePath())` lists first, resolved with `lymcwd.ResolveWorktree`.
  So the reader spawns no git and a tier0 caller whose location is no git worktree reads `false`.
- Any failure, a prime that does not resolve, a chain that does not read, or a stack that does not load or holds a refused value,
  reads `false` and is logged as a warning naming the worktree (`slug`, or the prime's name for `""`) and the cause.
  Reading `false` falls back to asking the operator, never to skipping one.
- The reader runs on every Beam-Await wake, Slack hook turn and halt notice, so a failure that repeats warns once:
  the warning goes through one package-level `logger.EpisodeSet` keyed by the worktree, `Fail` on a failed read and `Recover` on the next good one.

`doc.go` gains a paragraph naming `Autonomy` as the reader of beam.yaml's `autonomy` for runs and orchs.

The lookup's precedence is `dobbyengine`'s and `TestLookupTaskSettings` pins it, so `TestAutonomy` pins only what the reader adds.
It is tier1, in a new file of `hubgeom_test`, over one `hubforge.CopyHub` hub laid out as `TestTaskSettingChain` lays it:
a beam pair under the prime and a task worktree added from the beam pair.
Each row writes `autonomy` into the worktree files of the task, beam and prime (`configengine.ConfigFile(anchor, "beam")`) as it names,
and asserts the answer:

- task `true` under beam `false` reads `true`, and task unset under beam `true` reads `true` (D11 through the real chain);
- the beam's own name with beam `true` and prime `false` reads `true` (D2, an orch's reader starts at its own worktree, not its parent's);
- the prime's own name with the prime `true` reads `true` (the prime's name passed as `""`);
- a beam file holding `autonomy: "yes"` reads `false` for the task, and the captured log (`logcapture.Capture`) holds one warning naming the beam's worktree (D2's criterion);
  a second read of the same failure adds no warning;
- a location in a plain temp directory, no git worktree, reads `false` with a warning, the prime that does not resolve.

The two log rows capture the process-wide logger, so they run serially, which a comment at them names.

**ImpactSummary:** New exported reader with no existing caller; later cards call it from the halt notices, loom and drift wiring, PR-Review, Beam-Await, the Slack hook and the orch pointers.
