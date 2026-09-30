# loom start refuses a worktree with an unfinished merge

## Goal

`lyx loom start` must never put a driver to work on a pair that is mid-merge.
Today a hub merge-in that stopped on a conflict (a `UU` path, `MERGE_HEAD` present) was followed by `lyx loom start`, which spawned a driver onto the conflicted tree.
Today `start` checks no merge state at all before it spawns or resumes a driver;
the recipe's own Preflight row runs only at the head of the recipe, inside the driver, so a resumed run never re-checks.
After this task, `start` refuses on its envelope with a distinct kind and names the remedy: resolve, `lyx fabric merge-stage`, `lyx fabric merge --continue` (or `--abort`).

## Scope

In:

- A read-only, `l`-in, vocabulary-neutral mid-merge probe exported from `internal/fabricengine`, covering both a fabric-parked merge and foreign git-level merge state on either side of the pair.
- A refusal in `lyx loom start` (and so its root alias `lyx start`) at the point where the invocation would spawn a driver (either arm) or resume a parked ly-drive driver.
- A new envelope kind for that refusal, declared beside `shedrun.StartNotParkedKind`.
- Docs: `start`'s `Long` help text, the loom entry in `docs/overview.md`, and the list of `l`-in entry points in `internal/fabricengine/doc.go`.

Out:

- `lyx loom step`, `lyx loom run`, `lyx shed step`, and the recipe's Preflight row — unchanged.
- `internal/preflight` — no new `CheckID`.
- Any change to how batten, ly-drive, or the hub react to the new kind beyond what their existing non-retryable-failure paths already do.
- Auto-resolving or auto-aborting the merge; `start` only refuses.
- `CONSTRAINTS.md` and `manifest/roadmap.md` — no new invariant, and this is a bugfix, not a roadmap item.

## Decisions

- **The gate lives in `lyx loom start`, not in `internal/preflight` or the Preflight row.**
  Rationale: the harm is putting a driver to work, and `start` is the one verb that does that.
  The Preflight row runs once at recipe head inside the driver, so it cannot guard a resume, and its `worktree-clean` check already flags a `UU` path on a fresh run.
  `step` must stay unguarded because Landing's `mergeresolve` legitimately holds the pair mid-merge inside a step.

- **Refuse only when this invocation would spawn a driver or resume a parked one.**
  The trigger is `runDriverSpawnAndWait`'s existing decision: `mustSpawn` true (go or llm arm), or a live driver strand with the park marker present (the resume branch).
  A live, working driver (strand live with no marker, or the go arm's run lock held) is left alone and `start` proceeds to attach exactly as today.
  A live strand with no marker over a run halted at a hand-back is neither a spawn nor a resume, so it keeps its `driver_not_parked` refusal on a mid-merge pair;
  the merge check applies once that driver parks and a later `start` would resume it.
  Rationale: Landing's conflict-resolution session runs mid-merge with a live driver, and an operator re-attaching to it must not be refused.

- **The check runs after the strand/run-lock probe and before park-marker removal, the resume line, or any spawn.**
  It releases the bootstrap lock and reports on the envelope exactly like the surrounding failure returns.
  The seed-and-commit steps before it stay where they are: that commit is weft-only, idempotent, and runs on every `start` invocation already.

- **A new exported probe in `internal/fabricengine` answers "is this pair mid-merge, and how".**
  It takes `l *lyxcwd.Location`, is read-only (no `MutationRecord`), and returns a vocabulary-neutral classification with three states: not mid-merge, a fabric merge parked (the fabric merge-state record exists), or foreign merge state (a `MERGE_HEAD` or unmerged index entries on either side with no fabric record), plus the conflicted paths.
  The conflicted paths are in the pair's one-repo form, the same form a fabric merge verb's `conflicts` carries, so `lyx fabric merge-stage` accepts them.
  It opens the pair from `l` itself and reuses the existing internals — `mergeRecordExists` and the four probes `foreignMergeStatePresent` combines — rather than re-deriving them.
  A pair the probe cannot open, or whose record location cannot be resolved, is a probe error, never "not mid-merge".
  Rationale: `loomcli` is outside the Fabric Vocabulary Invariant's owner set and cannot name the sides, and `MergeStateActive` (weft-only) and `Fabric.MergeInProgress` (record-only) each miss the incident's shape.

- **One new kind, `merge_in_progress`, declared as a constant beside `shedrun.StartNotParkedKind`.**
  The envelope is `output.ErrFields` with `kind` and `conflicts` (the conflicted paths, an empty list when there are none).
  Rationale: an unattended caller (batten, a script, the hub) must tell this non-retryable refusal apart from the retryable `driver_not_parked` by kind, never by message text.
  Batten needs no change: its child-spawn error wraps the retry sentinel only for kind `driver_not_parked`, and its inner run returns every other spawn error as a hard error, so a `merge_in_progress` refusal halts rather than loops.

- **The message names the remedy by case.**
  Fabric merge parked: resolve each conflicted path, mark it resolved with `lyx fabric merge-stage <path>...`, then run `lyx fabric merge --continue` (or `lyx fabric merge --abort` to discard the merge), then re-run `lyx loom start`.
  Foreign merge state: a git merge, cherry-pick or squash that fabric did not start is in progress; conclude or abort it with git, then re-run `lyx loom start` — the fabric verbs refuse foreign state, so naming them there would send the operator into a second refusal.
  Rationale: the board names the fabric remedy, and the foreign case needs its own text because that remedy fails on it.

- **A probe error refuses on the envelope.**
  This covers a pair the probe cannot open or whose record location cannot be resolved.
  Rationale: an undetermined answer is not evidence the tree is clean, and spawning on it reopens the bug.

## Constraints

- Fabric Vocabulary Invariant: `loomcli` code, comments and identifiers never name the two sides; the probe's result type and its fields are vocabulary-neutral.
- Fabric Git Invariant: the probe goes through `fabricengine` and `gitrepo`, never raw git from `loomcli`.
- Mutation Record Invariant: the probe is read-only and embeds no `MutationRecord`.
- Driver Choice Single-Site Invariant: the refusal must not add a second read of the recorded seed driver; it keys on the existing spawn/resume decision, which is driver-agnostic.
- CLI/Cobra Invariant: the refusal is a pre-flight envelope failure, before the step-7 interactive handoff.
- Test Tier Purity and Hermetic Git Test Environment Invariants apply to the new tests; hub fixtures come from `internal/hubforge`.
- Semantic line breaks in every touched `.md` and multi-line comment.

## Auto-mode assumptions

- The incident's merge-in was run with `lyx fabric merge-in`, so the common shape is a fabric-parked merge; the foreign shape is covered because it is equally reachable and equally harmful.
- Refusing a `start` whose only merge state is a record left by a crashed Landing `mergeresolve` session is acceptable: the operator runs `lyx fabric merge --abort` (which `mergeresolve` would itself have done on re-entry) and re-runs `start`.
- Foreign merge state on the weft side counts as mid-merge, matching `foreignMergeStatePresent`'s two-sided form.

## Open risks

- A crashed Landing conflict session leaves a fabric record that now blocks `start` until the operator aborts it; the refusal text names `--abort`, but the operator cannot tell this origin from a hub merge-in by the message alone.
- A merge-in started into the worktree while a driver is actively stepping is not guarded; that race is outside this task.
## Acceptance criteria

- `start` over a pair with a conflicted `lyx fabric merge-in` parked refuses with kind `merge_in_progress`, lists the conflicted paths in `conflicts`, names `lyx fabric merge-stage`, `lyx fabric merge --continue` and `--abort`, exits non-zero, and spawns no driver on either arm.
- Same pair after `merge-stage` of every conflicted path but before `--continue` (record present, no unmerged entries) still refuses with the same kind and an empty `conflicts`.
- Foreign state refuses with kind `merge_in_progress` and the foreign-case message, for each shape: a conflicted plain `git merge` (both `MERGE_HEAD` and unmerged entries), a resolved-but-unconcluded `git merge` (`MERGE_HEAD` only), and a conflicted `git merge --squash` (unmerged entries only).
- A parked ly-drive driver (live strand plus park marker) over a mid-merge pair is refused, and no resume line is typed into its pane.
- A live, working driver (live strand, no marker, run running) over a mid-merge pair is not refused: `start` behaves exactly as today.
- On refusal the bootstrap lock is released and a stale park marker is left untouched.
- A clean pair's `start` behaves exactly as before; existing `start` tests stay green.
- The `fabricengine` probe has a unit/integration row per state: clean, fabric-parked with conflicts, fabric-parked with none, and each foreign shape on each side.
- A probe error surfaces as an error envelope and spawns nothing, including over a pair the probe cannot open.
- A live strand with no marker over a run halted at a hand-back, over a mid-merge pair, still refuses with kind `driver_not_parked`.
- Over a pair with conflicts, `conflicts` lists paths in the same one-repo form a fabric merge verb's `conflicts` carries.
- `start`'s `Long` text, `docs/overview.md`'s loom entry and `internal/fabricengine/doc.go` describe the refusal and the probe in the same commit.

## Notes for the plan writer

- The spawn/resume decision point is `runDriverSpawnAndWait` in `internal/loomcli/start.go`; the resume branch reads the park marker, and every failure there releases the bootstrap lock before `output.Err`.
- The existing fabric probes worth reusing sit in `internal/fabricengine/mergestate.go` (`mergeRecordExists`, `mergeBlocksMutation`, `foreignMergeStatePresent`) and `mergestateactive.go` (`MergeStateActive`, an `l`-in precedent).
- The fabric remedy wording already exists in `internal/fabriccli/envelope.go`'s conflict error; keep `start`'s text consistent with it.
- `internal/battencli/wire.go`'s `childNotParked` is where batten classifies a child `start` refusal by kind; `internal/battenshed/innerrun.go` retries only on `ErrChildNotParked`.
