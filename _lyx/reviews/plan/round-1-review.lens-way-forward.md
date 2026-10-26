```yaml
lens: way-forward
outcome: reported
```

**Lens: way-forward.** For each refusal, stop or recovery step the cards add: the next step it names, and whether that step can succeed from the state it leaves.

- Card 1, the task refusals of `judgeCreateEntry` and `judgeLandEntry`: make a project under a new slug, then `lyx beam create <that slug>`. Progresses.
- Card 1, `checkBeamPayload`'s beam key on a task: "making a project entry and `lyx beam create <slug>`". Ambiguous: if `<slug>` is filled with the task's slug, as today's `%s` is, create refuses the task. Finding.
- Card 1, the start guard's marked task pair: `lyx batten cancel <slug> --remove` from the prime, then release and run. Progresses.
- Card 3, `checkProjectFields` and `normalizeTask` recipe refusals: name the field fix. Progresses.
- Card 3, `checkOwnerPayload`: `lyx adopt <AdoptSubject>`. Progresses.
- Card 4, `namedEntries` refusing a task or grouping project: `lyx adopt <beam|root>`, or `lyx beam create`. Progresses.
- Card 4, the `takeOverSlug` stop: `lyx adopt <project> --keep-local|--drop-local`. Progresses.
- Card 5, `BeamRunLockedError`: `lyx batten cancel <slug>`. That cancel refuses while members remain and names them (card 20), then succeeds. Progresses over two hops.
- Card 7, `EnsureLive` refusals: `lyx adopt root`, `lyx adopt <slug>`, `lyx beam create <slug>`, `lyx batten status <slug>`. Progresses.
- Card 7, `ensureParentDobby` held: `lyx batten status <parent>`. Informational; the landing proceeds. Progresses.
- Card 7, `ensureParentDobby` start failure: `lyx beam start <parent>`. For a parent that is the prime the card names no `root` form. Finding (NIT).
- Card 10, `beamUpRefusal`: the owner case names `lyx adopt <slug>|root` and the adopt brings the beam up (card 24); the move case names the move's way forward; the held case names `lyx batten status`. Progresses.
- Card 11, `resolveBeamTarget`: the task-pair, unknown-slug, outside-hub and branch-on-origin refusals each name a step. Progresses.
- Card 11, a slug given with `--all`: the card names no way forward. Finding.
- Card 13, `BeamLandRefusal` blockers: "land it with `lyx batten run`" or cancel. Progresses at Beam-Await, where the hold is lifted.
- Card 14, `landHaltReason`, member with a pair or a run in flight: `lyx batten cancel --remove`, then `lyx loom resume`. Does not progress: the cancel leaves the member open, so the resume halts again. Finding.
- Card 14, member with no run status: `lyx board upsert` moving `project`, then resume. Progresses; checking the beam's `order` runs only when that order changes (`project.go:45`).
- Card 14, read error: `lyx loom resume`. Progresses.
- Card 15, `BeamUp` failed step: halting Stuck naming `lyx loom resume`. Progresses once the cause is fixed.
- Card 15, failed re-check: lift the hold, restart the dobby, post the blockers. Progresses.
- Card 15, failed restart: `lyx beam start <slug>`. Progresses, since the hold is lifted.
- Card 15, `Recheck` error or failed `StopDobby`: `lyx beam land <slug>`. Progresses.
- Card 15, interrupted re-entry at Parent-Align: `lyx beam start` and `lyx beam land`. Progresses.
- Card 15, re-entry after a halt or pause past the request's removal: names nothing and leaves the beam's dobby stopped. Finding.
- Card 17, `pawlRecheckLines`: `lyx adopt <slug>` or `lyx adopt <parentSubject>`. Progresses.
- Card 18, `memberHoldRefusal`: cancel the member from the parent worktree, then move its `project` off; or `lyx batten run <member>` once the beam is back at Beam-Await. Progresses.
- Card 18, grouping project: `lyx beam create`; absent parent pair: `lyx beam create <parent>` or `lyx adopt <parent>`. Progresses.
- Card 20, members remain: refuses with `BeamLandRefusal`'s text, whose land-it option cannot succeed while the pawl run holds the hold; the cancel option can. Finding (LOW).
- Card 20, unreachable origin: the reconcile's way forward and a re-run. Progresses.
- Card 20, cancelled beam: `lyx beam create <slug>` re-starts it from `draft`. Progresses.
- Card 21, every refusal (wrong worktree, absent parent pair, lost owner write, lost root claim, refused first pass): each names a step. Progresses.
- Card 22, every refusal (legacy beam, not at Beam-Await, owner or parent owned elsewhere, members): each names a step. Progresses.
- Card 23, the can-land notice: `lyx beam land` or `lyx adopt <subject>`. Progresses, per D12.
- Card 24, held beam: `lyx batten status` and `lyx adopt <parent>`; failed `BeamUp` or hold read: `lyx beam start <slug>`. Progresses.
- Card 25, owner ways forward naming `lyx adopt <subject>`, with the slug as a fallback. Progresses.

## Findings

1. MEDIUM, design, card 14 (`landHaltReason`). A member with a pair or a run in flight gets `lyx batten cancel <slug> --remove`, then `lyx loom resume`. The cancel writes the member back to `draft` (`clearRunClaim`, `internal/battencli/cancelcmd.go:461`) and leaves its `project` on the beam. `beamMembers` still lists it as an open member (`internal/battencli/beamland.go:103-130`), so the resume halts again at the same check. Card 18's `memberHoldRefusal` already names the follow-up board move "since a draft member still blocks the landing". Card 14 should name the cancel, then the `lyx board upsert` move off the beam, then the resume.

2. MEDIUM, design, card 15 (`BeamAwait.Call`, the "halted or paused after the request's removal" bound). The accept has stopped the beam's dobby. A run halted or paused between the request's removal and Done re-enters as an ordinary wait: it posts no notice and does not restart the dobby. The pass path may also have removed the marker already. The beam then sits at Beam-Await with its dobby down, so no member starts and no can-land notice goes out (card 23 posts from the dobby), and nothing names `lyx beam start <slug>`. Fix: on any entry with no request standing, ensure the dobby is live through `StartDobby` (idempotent `EnsureLive`), or post the interrupted notice. Evidence: card 15's Intent, the re-entry bullets.

3. LOW, consistency, card 20 (`refuseBeamCancel`). A beam with members is refused with `BeamLandRefusal`'s text at every pawl row. For a member with a pair, that text says "land it with `lyx batten run <member>`" (`internal/battencli/beamland.go:79`). Once the pawl run holds the hold (Parent-Align on, such as a Finalize halted on an open member), that landing is refused: by `ResolveBattenPrime` inside the beam (card 5), by Worktree-Create's `memberHoldRefusal` (card 18), and by `ensureParentDobby`'s hold refusal (card 7). D9 bars that wording at Finalize for the same reason. Only the cancel alternative progresses.

4. LOW, design, card 11 (`startCmd`). "refuses both together" for `[slug]` with `--all`, and the card names no way forward. `PATTERN-refusal-way-forward` wants one, such as dropping the slug or `--all`.

5. LOW, consistency, card 1 (`checkBeamPayload`). The way forward is "making a project entry and `lyx beam create <slug>`". Today's text fills `%s` with the entry's own slug (`internal/boardengine/project.go:82`). Filled the same way, it names `lyx beam create <task-slug>`, which create refuses for a task. The card's other task refusals say "under a slug of its own, then `lyx beam create <that slug>`"; this one should match.

6. NIT, consistency, card 7 (`ensureParentDobby`). The failure way forward is `lyx beam start <parent>`. For a run whose parent is the prime, the card does not say it names `lyx beam start root`, though it does say so for the owner refusal (`lyx adopt root`).

Outside this lens, for the handler: after card 25 moves `resumeRefusal`, `refuseStartOwnedElsewhere` and `claimBoardEntry` to `ownedElsewhereFor`, and card 3 deletes `resumeOwnerClaim`, `ownedElsewhereRefusal` has no production caller. Card 25 nonetheless keeps it "for a caller with no board read" (`internal/battencli/owner.go:151,201,269`, `internal/battencli/resumereconcile.go:128`).

Tool calls:
- `internal/battencli/beamland.go:59-132` and the order lines of `project.go`: the land refusal's exact member wording and whether a move off a project touches the beam's `order`. The manifest held neither.
- grep for `ownedElsewhereRefusal(` callers: to check which callers keep the slug-only fallback. Not in the manifest.
- awk over `owner.go` and `resumereconcile.go`: to map those call lines to their enclosing functions. Not in the manifest.
