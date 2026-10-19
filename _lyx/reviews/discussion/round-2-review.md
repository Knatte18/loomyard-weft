---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:54
    summary: D3 and its acceptance criterion claim a boot marks a prime session an older binary created, but boot never re-pins an already-up session, so only the attach pre-flight can mark it
    fix: In D3 (decision-record.md:54), replace "so a session that already existed before the new binary gets it on its next boot or attach" with a statement that a boot marks only the session it spawns, and a prime session that was already up when the new binary arrived is marked at its next attach pre-flight; add the bound that until a lyx attach to the prime, such a session draws in the second loop. In Acceptance criteria (decision-record.md:167), replace "is marked after reed boots onto it and after an attach to it" with "is marked after an attach to it, and a prime session a fresh boot spawns is marked by that boot".
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:42
    summary: D2 says the three probes go through runChecked, but it describes four git reads, so the channel of the tree comparison against the first parent is left unstated
    fix: In D2 (decision-record.md:42), replace "The three probes are CLI calls through `runChecked`" with "The four probes, the three stopped-empty conditions and the comparison of the picked commit's tree with its first parent's, are CLI calls through `runChecked`".
---

### [MEDIUM] D3's boot-time mark cannot reach an existing prime session

**Location:** `_lyx/discussion/decision-record.md:54`, `_lyx/discussion/decision-record.md:167`

**Issue:** D3 says the mark is set wherever `pinGeometryOptionsLocked` runs for a prime geometry, "at boot and in the attach pre-flight, so a session that already existed before the new binary gets it on its next boot or attach".
The acceptance criterion repeats it: "A prime session created by an older binary is marked after reed boots onto it and after an attach to it".
In the code, the boot path in `internal/reedengine/lifecycle.go` returns `false, nil, nil` as soon as `sessionSubstrateLocked` reports the session up and usable (lifecycle.go:378-381), before the spawn loop and before `pinBootOptionsLocked`.
`pinBootOptionsLocked`'s own doc says "Boot options never re-apply to an already-up session, which is why the attach pre-flight re-pins the geometry ones itself" (lifecycle.go:557).
So a boot only pins, and would only mark, a session it has just spawned, which the new binary created.
A prime session an older binary created is marked only by the attach pre-flight (`attach.go:97`), and the criterion's "after reed boots onto it" names a state no test can reach.
The false premise also hides a visible bound: until someone attaches to the prime through lyx, a client on a pair session sees the prime's button in the second loop, in name or id order, not first.
D3 should state that and drop the boot claim.

### [LOW] D2's probe count leaves the tree comparison's channel open

**Location:** `_lyx/discussion/decision-record.md:42`

**Issue:** D2 names three stopped-empty conditions (a pick in progress, no unmerged entry, the index equal to HEAD), then adds a fourth read, the picked commit's tree against its first parent's.
The next sentence, "The three probes are CLI calls through `runChecked`", covers only the first three.
A plan writer cannot tell whether the tree comparison is also a CLI read through `runChecked`, or a go-git read under `PATTERN-gitrepo-client-boundary`, which assigns local reads to go-git and `dotgit`.
The count is also a pinned tally that already disagrees with the Decision's own text.

## Focus verdicts

- D4's spawn logging: `go list -deps ./internal/logger` gives `dotgit`, `lyxcwd`, `lyxdirs`, `proc` and `logger`, with no `gitexec`, so gitexec importing `internal/logger` closes no cycle.
  The allowlist entry in `cmd/lyx/spawnobservability_test.go:72` rests on a reason that no longer holds, as D4 says, and reed's probe logs through `internal/logger`, which reedengine already imports.
  Not a defect.
- D4's git coverage: the production `exec.Command` sites reachable from `cmd/lyx` that run git are `internal/gitexec/gitexec.go` and quarry v0.2.1's `internal/gitsrc/gitsrc.go:336`, reached only through `(*quarry.Repo).DeltaGit`, whose one caller is `internal/planglyph/delta.go:36`.
  D4 names that path and bounds `internal/gitkit`.
  `internal/hubforge` also spawns git directly but is absent from `go list -deps ./cmd/lyx`, so the Rationale's claim about "every git command lyx runs" holds.
  Not a defect.
- D4's cache against test hooks: tmux hook-built runners carry their own result, and gitexec has no exec hook (its only test seam is `SetRemoteDeadlineForTest`), so the cache is seeded only by the named serial tests.
  Go holds a package's `t.Parallel` top-level tests until its serial top-level tests have finished, so the injected line cannot leak to a parallel test.
  Not a defect.
- D2's empty-from-start commit: on this host's git 2.53.0, both a plain `git cherry-pick` and `git cherry-pick --empty=drop` of a commit empty from the start exit 1 with `CHERRY_PICK_HEAD` present and HEAD unmoved.
  So such a commit passes the stopped-empty test, the tree comparison is what catches it, and the result matches today's.
  Constraints ("an empty-from-start commit refused as before") and the acceptance criterion at decision-record.md:164 agree with D2.
  Not a defect, apart from F2.
- D3's order and mark sites: `status-format[1]` is pinned only in `pinGeometryOptionsLocked` (`windowsize.go:67`), called from `pinBootOptionsLocked` (lifecycle.go:582) and the attach pre-flight (attach.go:97), so D3 covers every pin site.
  D3 and Constraints promise name order on tmux 3.4, which is the order of tmux 3.4's name-keyed session tree.
  I could not render it on this host: a private tmux 3.6 server under `.scratch/` exited at once in this sandbox.
  The boot site cannot reach an existing session (F1).
- D6's marker: `Delivery.Paused` is called only by the worker's `locate` (`deliver.go:242`) and by `Store.interruptTarget` (`verbs.go:483`).
  `locate` reads the pause after `Strands` and before `probe` takes the pane lock and snapshots the store.
  So a pause read for the receiver's strand that starts after the post returned implies a snapshot after the post, and D6 handles the interrupt reader.
  Not a defect.
- D7's seam: `launchBlockedOnLock`'s callers launch `MergeIn`, `Merge` and `MergeAbort`, which take the weft write lock at `merge.go:195`, `merge.go:399` and `mergelifecycle.go:382`, all inside D7's named files.
  The slowed reproduction is a one-off diagnostic that never lands, and no acceptance criterion needs a slowing seam.
  Not a defect.
