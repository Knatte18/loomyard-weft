```yaml
lens: fix-claims
outcome: reported
```

Checked every bound, behaviour and code fact the fix added or rewrote against the code at HEAD.

Claims checked:
- D4: `StartTied` plus `Tied.Kill` kill the whole tree, through a process group on Linux and a kill-on-close job on Windows. **Holds** (`internal/proc/tied.go:33-37,80`, `tied_linux.go:18`, `tied_windows.go:50`).
- D4: the gate's `runTiedChild` ends a timed-out run. **Holds** (`internal/gatecli/build.go:36-47`).
- D4: `.lyx/` is already excluded through `.git/info/exclude` at clone. **Holds** (`internal/fabriccli/clone.go:113-116`, `fabriccli/doc.go:10`).
- D4: the plan gate's type load is a slot holder. **Holds** (`internal/gateslot/doc.go:36-37`).
- D6 and D2: a reason opening with `DecisionHaltPrefix` is marked by `arbiterstop.HandsBack` and handed back ahead of the tackler and the arbiter. **Holds mechanically**:
  - `arbiterstop.go:61-63`;
  - `sheddriver/doc.go:30` puts hand-back ahead of every arbiter and tackler row;
  - `shedengine/run.go:394-400` keeps the Stuck reason on a blocked stop;
  - `loomcli/drivewiring.go:292`.
- D5: an adopt ends the old machine's processes, and `.lyx/drift/` is never tracked. **Holds** (`tendercli/deps.go:266-269`).
- D3: Plan-Write's fresh spawn archives the plan. **Holds** (`loomshed/planwrite.go:123-126`), as the review cited.

Findings:

- **LOW.** Concerns D6's new sentence that a scope-`full` halt over drift "has a reason opening with `shedadapters.DecisionHaltPrefix`", and D2's "a decision halt for the parent (D6)".
  Summary: the record reuses a constant whose own doc gives it one narrower meaning, and it does not say that doc must widen.
  Evidence: `internal/shedadapters/bouncerdecision.go:25-26` says `DecisionHaltPrefix` "opens the Reason of the halt a recorded accept over a held refusal meets at the budget and the hard cap", and that "a caller that must tell that halt from any other Stuck reads the Reason for it".
  A Drift-Verify reason carrying the prefix would make that doc false, and any reader that takes the prefix to mean a bouncer decision halt would be wrong.
  The hand-back claim holds, but the record should say the constant's doc, or a shared hand-back marker, widens to cover Drift halts.

No other findings.

Tool calls:
- Grepped for the `.git/info/exclude` writer and for the `HandsBack` and `sheddriver.Deps` wiring. My context had the clone comment and the arbiterstop doc, but not whether the exclude writer is shared or which drivers wire `HandsBack`.
- Grepped the gatecli timeout path and read `shedadapters/bouncerdecision.go:20-30`. My context had `runTiedChild`, but not a timeout caller, and not the `DecisionHaltPrefix` doc.
