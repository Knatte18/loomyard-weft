---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/19-locationkit-callers.md
    summary: card 19 deletes newPortalLauncherTestLocation but omits three fabricengine files that call it, so the package stops compiling after the card
  - id: F2
    severity: BLOCKING
    location: _lyx/plan/44-slices-cleanup.md
    summary: card 44 deletes reedengine's equalStringSlices but omits strand_test.go and lifecycle_test.go, which call it, so reedengine tests stop compiling
  - id: F3
    severity: MEDIUM
    location: _lyx/plan/25-shedfake-kit.md
    summary: the Run-only scripted shuttle fakes in mergeresolve and frictionengine are not cycle-forced, yet no card converts them and the plan records no reason they stay
  - id: F4
    severity: NIT
    location: _lyx/plan/22-shuttlefake-cli.md
    summary: card 22's ImpactSummary misses smoke_parentreview_test.go, whose parentNoticeEngine embeds the gateRepromptReadEngine the card rewrites
---

### [BLOCKING] Card 19 leaves fabricengine uncompilable

**Location:** `_lyx/plan/19-locationkit-callers.md`

**Issue:**
Card 19 deletes `newPortalLauncherTestLocation` from `internal/fabricengine/portallauncher_test.go`, but at HEAD the helper is also called from `internal/fabricengine/hubworkspace_test.go:20`, `launcher_content_test.go:192` and `origin_test.go:82,109,128`, all `package fabricengine` and untagged.
None of the three is in the card's Edit list, so after the card the package's tests no longer compile, against D12's rule that every step leaves the suite green.
`origin_test.go`'s header comment (line 2) also names the deleted helper and would go stale.
The ImpactSummary's "Three untagged test files" undercounts the blast radius accordingly.

**Fix:**
Add the three files to card 19's Edit list, convert their calls to `locationkit.Location`, reword `origin_test.go`'s header to name the kit, and rewrite the ImpactSummary to cover both packages' six files.

### [BLOCKING] Card 44 leaves reedengine uncompilable

**Location:** `_lyx/plan/44-slices-cleanup.md`

**Issue:**
`equalStringSlices` is defined in `internal/reedengine/reconcile_test.go:15` and also called from `internal/reedengine/strand_test.go:1086` and `lifecycle_test.go:328`.
Card 44 edits only `reconcile_test.go` in reedengine, so deleting the definition breaks the other two files.
The ImpactSummary's "Five test files" misses them.

**Fix:**
Add `reedengine/strand_test.go` and `reedengine/lifecycle_test.go` to card 44's Edit list and its ImpactSummary.
The copy is a plain length-then-element comparison, which `slices.Equal` reproduces, nil and empty included.

### [MEDIUM] The mergeresolve and frictionengine shuttle fakes are neither converted nor justified

**Location:** `_lyx/plan/25-shedfake-kit.md`, `_lyx/plan/29-producers-shedfake.md`

**Issue:**
`internal/mergeresolve/mergeresolve_test.go:93` and `internal/frictionengine/reflect_test.go:22` each define a `fakeShuttle` over the one-method seam `Run(shuttleengine.Spec) (shuttleengine.Result, error)`: per-call scripted results and errors, a recorded spec list, and (mergeresolve only) a `runHook(callNumber, spec)`.
frictionengine's copy says it follows mergeresolve's pattern, so it is the same fake copied.

The decision record requires their conversion.
The acceptance criterion allows a consumer package its own merge-shuttle fake only in D1's cycle-forced cases, and D1 counts even a seam's own defining package as a kit user unless a cycle bars it (it names `shuttleengine`'s own tests as cycle-forced).
Neither package is cycle-forced: `go list -deps` over shedfake's imports (`shuttleengine`, `burlerengine`, `shedengine`, `websterengine`) reaches neither `mergeresolve` nor `frictionengine`.
The seam shape is faked in at least six packages, well past D1's two-package threshold, and a `shuttleengine.Run` signature change today edits both copies on top of the kit.

The plan neither converts them nor records why they stay.
Card 25's `MergeShuttle` (zero `Result`, nil error, optional `RunFn`) also cannot reproduce mergeresolve's copy without a local scripting closure, against D1's bound that the kit reproduces the richest copy it replaces.

**Fix:**
Extend card 25's `MergeShuttle` to reproduce mergeresolve's copy: scripted `Results`/`Errs` by call order (a zero value once exhausted, as both copies answer), recorded `Specs`, a `DuringRun(callNumber, spec)` hook, and the existing `RunFn` override.
State that it also satisfies `frictionengine.Shuttle` structurally.
Add `mergeresolve/mergeresolve_test.go` and `frictionengine/reflect_test.go` to card 29's Edit list and convert both fakes there.
Update card 29's Intent, ImpactSummary and its Card Index line to match.

### [NIT] Card 22's ImpactSummary misses a dependent smoke file

**Location:** `_lyx/plan/22-shuttlefake-cli.md`

**Issue:**
`internal/loomcli/smoke_parentreview_test.go:36-39` defines `parentNoticeEngine`, which embeds `gateRepromptReadEngine` and builds it as `gateRepromptReadEngine{quietSeconds: 3}`.
Card 22 rewrites `gateRepromptReadEngine` to embed `shuttlefake.Engine`, so `parentNoticeEngine` must still construct and satisfy `shuttleengine.Engine` afterwards (a pointer-receiver kit embedded in a value type would break that).
The ImpactSummary names only "two loomcli smoke files".

**Fix:**
Name `smoke_parentreview_test.go` in card 22's ImpactSummary as the dependent embedder whose construction must keep compiling, and add it to the Edit list in case the embedding forces a change there.
