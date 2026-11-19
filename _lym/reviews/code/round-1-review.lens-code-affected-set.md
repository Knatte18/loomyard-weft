```yaml
lens: code-affected-set
outcome: reported
```

Task: review the diff through the code-affected-set lens, listing what each change forces outside the lines it touches.

## Per change: what it affects, and whether the diff covers it

- **`gateslot.RedFirstCall` and the two exit codes:** read by `shedrecipe` (entries_gate.go:335), `loomcli` wiring (wiring.go:971), `loomshed.redFirstFailure`, the gatecli verb, and the docs in `gateslot`/`gatecli`/`loomshed`/`overview.md`. Covered.
- **`gateslot.PackagesFailedOutsideTests`:** read by `judgeRedFirst`, with docs in gateslot doc.go:312. Covered.
- **`verbDiffBase` replacing `mutateRequest.diffBase`:** read only by `runMutate` and `runRedFirst`; no test refers to the old method. Covered.
- **`testreach.ChangedTests` and `declaredFuncsWhere`:** read by `changedTestsAt`, and `declaredFuncs` keeps its behaviour; testreach doc.go updated. Covered.
- **New `shedrecipe.Env` seams `EntryLabels` and `RedFirstCommand`:** filled by `loomcli` wiring, `envkit.FullEnv` and the `shedrecipe` fixture; the batten `intentionallyNil` map is updated. Covered.
- **`red-first` in the gate vocabulary:** read by the `resolvePlainGateClosure` error list, its tests, `shedrecipe` doc.go, the darn recipe and `TestDarnVerifyBudget`. Covered.
- **New `verifytree.Verify` site `internal/loomshed/redfirstgate.go`:** the site list in `cmd/lym/verifiedtree_test.go`, the "Verified tree" section of `pattern/PATTERN-gate-slot.md`, and the `verifytree.Verify` doc comment. **Not covered** (finding 2).
- **`NoTestSeamTitle` and `BugLabel`:** `PATTERN-code-conventions` "Named constants", and the repeated literals in gatecli/redfirst.go:281, reportengine/landings.go:54, adoptcli/fixloop.go:42 and selfreportengine/selfreport.go:32. **Not covered** (finding 1).
- **Code-Review rubric bullet "Root cause of a bug fix":** read by every loom recipe's Code-Review, tapestry as well as darn. Only the darn writer stencil asks for the trailer. **Partly covered** (finding 3).
- **Darn write stencil "A bug entry":** checked against the parent directive and the no-denied-recovery rule. It names `lym creel post --parent`, which the writer may run, and gives `lym loom decision add` to the parent. Covered.
- **Commit messages:** no `Pattern-Departure:` or `Loosened-Check:` trailer is needed. The changed expectations in `TestDarnVerifyBudget` and `TestResolveGateSpec_*` add rows and loosen nothing. Covered.

## Findings

1. **BLOCKING, design.** Location: internal/gatecli/redfirst.go:281 and internal/loomshed/redfirstgate.go:21,24.
   `PATTERN-code-conventions` "Named constants" requires one constant declared once and never a repeated literal, and two names break it:
   - The `"No test seam"` title is declared as `loomshed.NoTestSeamTitle`, but the gatecli refusal spells it again as a literal.
   - `BugLabel = "bug"` is a further declaration of a label name already declared at reportengine/landings.go:54 (`labelBug`), adoptcli/fixloop.go:42 (`fixLabelBug`) and selfreportengine/selfreport.go:32 (`defaultLabel`).

   Fix: declare both once in a package both sides already import, such as `gateslot`, and reference them from there.
   Evidence: pattern/PATTERN-code-conventions.md "Named constants"; redfirst.go diff line 281; the grep of `"bug"` in the manifest.
2. **MEDIUM, consistency.** Location: cmd/lym/verifiedtree_test.go:19–26, pattern/PATTERN-gate-slot.md "Verified tree", and internal/verifytree/verify.go (doc of `Verify`).
   The diff adds a new `verifytree.Verify` site but leaves the documents and tripwire that describe the sites wrong:
   - The tripwire's `planVerifySites` does not list redfirstgate.go, so nothing guards that it keeps its `Hold`.
   - The pattern background still says every verify site runs a gate.yaml-rendered `lym gate test --wait` call, and it does not list the Darn-Write red-first gate among the sites.
   - `Verify`'s doc says "The command is one `lym gate test --wait` call, or such calls joined by `&&`", which a `lym gate red-first` command now contradicts.

   Fix: add the file to `planVerifySites`, and update the background and the doc comment to name the red-first call.
   Evidence: verifiedtree_test.go:19–26; PATTERN-gate-slot.md "Verified tree" bullets; verify.go:309–310.
3. **MEDIUM, consistency.** Location: contracts/stencils/loom/loom-rubric-code-review.md:87–90.
   The new bullet makes it a finding when "the answer key asks to fix a bug and no commit of the range carries the trailer". The rubric is shared by every loom recipe's Code-Review, but only the darn writer stencil (darn-template-write.md:107–119) asks for a `Root-Cause:` trailer.
   As a result, a tapestry run of a bug entry draws a finding whose request its writers never received.
   Fix: either limit the missing-trailer clause to runs whose writer is asked for the trailer, or add the ask to the tapestry writer stencils.
   Evidence: the rubric diff lines 87–90 and the darn stencil diff.

No PATTERN changes were declared, so none had to land.

Tool calls:
1. Grepped for other lists of gate verbs, callers of `diffBase`, the verified-tree tripwire, the refusal spec's exemptions and the `verifytree.Verify` callers, none of which the manifest held.
2. Read `cmd/lym/verifiedtree_test.go`'s site list and the other mentions of `gate mutate`, to see whether the new site or verb is missing from them; the manifest held only grep hits for these.
