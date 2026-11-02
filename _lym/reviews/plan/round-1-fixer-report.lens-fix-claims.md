```yaml
lens: fix-claims
outcome: reported
```

Task: check every claim the fix added or rewrote against the code at HEAD (fix-claims lens).

**MEDIUM: card 9's time bound for the end is wrong**
- Changed text: card 9, Bound paragraph: "The end runs under the run lock, and in `Relaunch` under the status lock too, for at most `runproc`'s grace and kill bound."
- Summary: the real bound is several times one grace plus one kill bound.
- Evidence:
  - `endGateFixers` calls `end` once per role of `gatefixer.Roles()`, and there are two roles.
  - `runproc.EndRole` re-lists the owner up to `endPasses` times. Each pass waits up to `endGrace` and then up to `killBound` (`internal/runproc/doc.go:17-20`; `internal/runproc/end.go:20-24`: `endGrace = 5s`, `killBound = 2s`, `endPasses = 3`).
  - So the worst case is roles × passes × (grace + kill bound), not one grace plus one kill bound.
- Fix: name `runproc.EndRole`'s grace, kill bound and passes for each fixer role, or drop the "at most".

Every other claim I checked holds at HEAD:
- `arbiterDepsFor` sets `GotoGuard: c.settledGuard()` (`internal/loomcli/arbiter.go:140`, inside `arbiterDepsFor`).
- `TacklerDeps.GotoGuard` only checks a verdict's goto (`internal/sheddriver/tackler.go:413`).
- `specFor` sets `GotoGuard` only in its location branch (`internal/loomcli/arm.go:464`).
- `Goto` runs the guard after its running, done and target checks and before its write. `Relaunch` runs it only for a non-empty `Target`, before `Admit`. `checkGuard` turns a guard error into a refusal that writes nothing (`internal/shedengine/goto.go`).
- Batten shares both consumes (`internal/battencli/drive.go:295,527`).
- Card 3's listed comment sites match the grep: `render.go` lines 1, 4, 25, 32 and 35; `strand.go:1`; `recoverbatch.go:1`; `template_test.go:375`; `doc.go:423,560`; `classify.go:22`; `stencils.go:276`.
- Card 7's spec paragraph is at `contracts/specs/shed-recipe-spec.md:72`.

Tool calls:
1. Read `arbiter.go:140`'s enclosing function and `runproc`'s end constants. My context had neither the function that holds the arbiter's `GotoGuard` nor `runproc`'s grace, kill and pass values.
