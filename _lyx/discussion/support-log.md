# Support log — batten-start-speed

## Interview

### Turn 1 — exploration (autonomous; no operator)

Read the board entry and the related entries `row-entry-gate`, `preflight-weft-push-sync`, `parallel-batch-parts`, `weaver`, `implementer-seam` and `regression-baseline-verify`.
Read batten's and loom's status seams, the shared `statuscommit` core, fabric's detached push spawn, `PushAnchored`/`PushPairAnchored`, add's origin probes, remove's archive, the switched producer, its registry entries, batten's recipe and template, the Land row's deploy runner and configengine's categories.
Findings that settled questions without asking:

- `CategoryExecutable` keys are refused in the local layer, which is where `run_settings` lands, so the board's open trust question has an existing answer.
- `Fabric.Commit` already spawns the detached push; `CommitAnchoredPaths`, which the seams use, does not.
- `SpawnPairPush` must not run in the prime; `PushPairAnchored` already pushes records only there.
- Add's `SkipGit`/`SkipPush` path is the no-probe path a local add needs; remove archives whenever the weft has an origin.

### Turn 2 — design batch (auto-picked)

Q1 status push: detached spawn per transition (recommended) vs. keep synchronous with a shorter lock wait vs. push only at end of the start rows. Picked detached.
Q2 Command trust: separate switch and command keys with categories (recommended) vs. empty command as the only off switch. Picked separate keys.
Q3 Command modes: sync only (recommended) vs. sync and async. Picked sync only.
Q4 Command failure policy: warn only (recommended) vs. warn and halt. Picked warn only.
Q5 local pair mark: per-branch git config (recommended) vs. a file under the pair's `.lyx` vs. no mark and a remove flag. Picked git config.
Q6 local remove guard: refuse an uncontained tip without `--force` (recommended) vs. no guard vs. a local archive tag. Picked the containment guard.
Q7 placeholders: `{lyx}`, `{slug}`, `{task_dir}` (recommended) vs. a wider set. Picked the three.
Q8 measurement: trace timestamps and status history into a research report (recommended) vs. new timing spans. Picked the report.

### Turn 3 — lens review (six forks, all returned)

- scenarios: fresh local pair's weft tip is the provenance commit, so the guard always refused — fixed, provenance commit exempted.
- scenarios: detached push races teardown — fixed, Remove waits for and holds the pair's push lock; acceptance row added.
- scenarios: hung children unbounded — fixed, bound by the absorbing lock, built in this task if absent.
- scenarios: tests could fork real push children — fixed, spawn injected; acceptance row.
- scenarios: `--local` over an existing branch undefined — fixed, refuses with way forward.
- scenarios: placeholder grammar collides with `${…}` — fixed, only three placeholders recognised, rest literal.
- scenarios: setting error lacks way forward — fixed; PATTERN-refusal-way-forward added to Constraints.
- scenarios, goal-scope: 100 ms target unverifiable and Open-Window unchecked — fixed, criterion on the integration-tier drive, Open-Window included; live numbers assigned to the orch in Open risks.
- scenarios: GUI children must outlive the row — fixed, verbs detach.
- scenarios: empty command under `--window` silent — fixed, Warn.
- scenarios: resume across the change not concrete — fixed, marker row named.
- premises: shared push-lock premise false (lock per records worktree) — fixed, assumption reworded, cause left to D2.
- premises: `StatusPushLockWait` keeps `commit-records` — fixed.
- premises: mark skip should sit lower — fixed in D3 and the note.
- pattern-fit: Windows quoting mismatch (`internal/shell` is pwsh, `verifyrun` uses cmd) — fixed, quoting beside `verifyrun`'s shell; PATTERN-shell-mechanics-seam citation dropped since it governs pane shells.
- pattern-fit: destruction chokepoint not named — fixed in Constraints.
- pattern-fit: new status-push entry overlaps PATTERN-push-both-sides — fixed, now a change to it.
- pattern-fit: log path derivation unclear — fixed, told by the owner; PATTERN-told-geometry added.
- pattern-fit: teardown log missing — fixed, start and end logged.
- pattern-fit: gitexec entries not named — fixed in Constraints.
- board-fit: local pair's portals, launchers and workspace entry undefined — fixed, none created.
- board-fit: `pair-from-parent` dependency unnamed — fixed in Out and Open risks.
- board-fit: more pushes race the board sync — fixed, Open risks.
- board-fit: `PATTERN-command-row` text locks in the switch — fixed, reworded.
- board-fit: async and halt should extend `Command` — fixed in Out.
- board-fit: detached push under a hub pause — fixed in D1's bound.
- board-fit: `implementer-seam` dependency — rejected, it orders work only and takes nothing from this task.
- board-fit: share placeholder expansion with `reed-status-buttons` — rejected, no consumer yet.
- goal-scope: measurement window started after Worktree-Create — fixed.
- goal-scope: D2's cut open-ended — fixed, bounded to batten's per-transition path.
- goal-scope: sync and warn-only stated only in the support log — fixed in Out.
- goal-scope: Preflight exclusion needs a reason — fixed.
- goal-scope: D3 adds more than licensed — rejected, the mark and guard are what make a local pair stay local and safe to remove.
- leanness: duplicated likely-cause sentence — fixed, removed from D1.
- leanness: Constraints restate Decisions — fixed, trimmed.
- leanness: D4 rationale repeats Goal — fixed, cut.
- leanness: call-site note in Notes — fixed, cut.
- leanness: vague IDE verb criterion — fixed, made concrete.

## Rejected alternatives

- Synchronous push with a shorter lock wait: still costs a network round trip per row.
- One push after the start rows: still synchronous, and a crash before it leaves the status unpushed until the next transition anyway.
- Empty command as the only off switch: a task's `run_settings` could then no longer switch a row per task, since the command key must be `CategoryExecutable`.
- Letting `run_settings` set a command: a board entry would run code on the machine, past the trust class `deploy` holds.
- An async `Command` mode: its only consumer is `Job-Start`, which is out of scope.
- A `halt` failure policy: no row in scope halts; `deploy` and `Job-Join` are out.
- A local-pair mark as a file under `.lyx`: outlives a branch deleted by hand and needs its own cleanup; git config dies with the branch.
- A remove flag instead of a mark: a later commit's detached push would still publish the pair.
- A local archive tag on remove: accumulates in the shared records repository and needs its own prune.
- New timing spans in the logger: trace timestamps already order every step.
- Taking the shed-wide entry gate now: note `row-entry-gate` owns it and the board entry puts it out.
- A separate `PATTERN-status-push-detached` entry: it overlaps PATTERN-push-both-sides, which now carries the rule.
- Quoting placeholders through `internal/shell`: it quotes for the pane shell (pwsh on Windows), not the `cmd` that `verifyrun` runs.
- Erroring on unknown brace placeholders: collides with shell syntax such as `${HOME}`.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 status push — resolved, auto-pick: detached spawn per committed transition.
- Q2 Command trust — resolved, auto-pick: switch key `CategoryNone`, command key `CategoryExecutable`.
- Q3 Command modes — resolved, auto-pick: sync only.
- Q4 failure policy — resolved, auto-pick: warn only.
- Q5 local pair mark — resolved, auto-pick: per-branch git config key owned by fabric.
- Q6 local remove guard — resolved, auto-pick: refuse an uncontained tip without `--force`.
- Q7 placeholders — resolved, auto-pick: `{lyx}`, `{slug}`, `{task_dir}`, shell-quoted.
- Q8 measurement — resolved, auto-pick: research report from trace timestamps and status history.
- Q9 verb homes — resolved, auto-pick: `lyx ide seed`/`lyx ide open` in `idecli`, the terminal window verb in `reedcli`.
- Q10 teardown against an in-flight detached push — resolved, auto-pick: Remove waits for and holds the pair's push lock.
- Q11 `--local` over an existing branch — resolved, auto-pick: refuse, naming a plain add.
- Q12 placeholder quoting on Windows — resolved, auto-pick: quote for the shell `verifyrun` runs, beside its choice.
- Q13 local pair's portals and launchers — resolved, auto-pick: none created.
