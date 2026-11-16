```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: checking that every bound, behaviour or code fact the fix added holds against the code at HEAD.

**MEDIUM — D10's new sentence "One fixer runs per failed verify evaluation, and the fixer's own gate calls spawn none."**
D6 can start a second fixer for the same evaluation, which breaks this bound.
D6 still says "That retry applies to each attempt, D7's re-entry attempt and D10's fixer strands included". So a fixer lost to the machinery gets a fresh fixer on the same rung, and nothing runs the verify again first.
That is two fixers for one failed evaluation, against the claim and against PATTERN-gate-slot's rule ("one fixer per failed evaluation; Go's re-run of the gate judges").
Only D12's crash path ("runs the verify again before any new fixer spawns") re-evaluates before the next fixer starts.

**LOW — D6's new line "the row returns Stuck naming the strand's guid and shuttle's way forward, `lym reed remove <guid>`".**
Shuttle's stop verb names no way forward when the removal fails.
`removeStrandIfPresent` in `internal/shuttleengine/stop.go:142-155` returns "shuttle: stop strand %s: remove it: %w" with no way forward.
Only `removeSupersededStrands` in `attach.go:241` names `lym reed remove`, and Webster's gatefixer `Await` turns a failed stop into a bare warning (`await.go:97-98`).
The text should present `lym reed remove <guid>` as Weaver's own way forward, not shuttle's.

**LOW — D2's new line "a probe that finds none removes, as `ProbeGated` does, only the strand of a run shuttle already holds ended".**
Superseded-strand removal also removes the strand of a run whose record still reads running.
`removeSupersededStrands` (`attach.go:233-245`) removes every respawn-eligible candidate whose strand is tracked and `strandRemovable`.
That set includes a tracked own-window strand with a present but dead pane, or one with a cleared binding that the age rule passed. Its record still reads `running` (`attach.go:553-564`, `leftoverThenAgeVerdict`, `stop.go:135-137`).
Its agent is gone, but shuttle does not hold its run as ended.
More accurate: "only the strand of a run whose agent shuttle judges gone".

**LOW — D12's new line "A fixer number state records with no `gatefixer` record is a crash between spawn and record".**
That state also arises without a crash between spawn and record:
- a crash before `Start`;
- a `Start` error, which `SpawnOrAttach` returns as transient with nothing saved (`spawn.go:110-111`);
- `abandon`, which stops the strand after a failed resolve, register or save (`spawn.go:115-137`).

Each of these is then counted as a machinery loss, although no fixer ran.
The handling stays safe because the probe finds nothing live, but the stated cause is too narrow.

**LOW — D6's new line "A fixer's strand is stopped by `gatefixer.Await` before it returns".**
`Await` stops the strand only on a terminal result.
A wait that runs out first returns `StatusRunning` and stops nothing (`record.go` `StatusRunning`; `doc.go` "Every terminal result stops the strand").
The claim holds only if Weaver awaits until the result is terminal, and the record does not say it does.

Further calls:
- grep of `ArchivedWebsterDirs` in `internal/loomshed/generation.go`: checks D2's claim that Describe's prior-generation readers do not list the reset archive. That reader was not in context. The claim holds, since it reads only `rework round/prior-generation/webster`.
- grep of gate handling in `internal/shuttleengine/wait.go`: checks how an attached gated Wait judges a finished run. That file was not in context. Finalize evaluates the gate once more, so D6's "judged through the attach's result" holds.
