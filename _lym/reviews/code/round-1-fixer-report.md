---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: The proposed rerun said to use the property the report's line names, but the line names none (internal/gatecli/tierrun.go:486); it points at the project file's IsTestProject condition. The step now says that, and sends the rerun through the gate only when the tier's command holds {args}.
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
---

# Round 1 fixer-report

## What changed

- F1: step 3 of `darn-fragment-make-change-gated.md` runs the measuring class through `lym gate test --gate darn-write <target> -- <args>`; the tier wording is gone, and the one-class, run-once limit stays. It also says the args reach every tier the key lists.
- F2: in `internal/gatecli/run.go` a tierless `--gate` call now sets the root, runs the key's lints, prints the failure report and closing line, and exits 1 on a finding, else 0. The notice reads "runs no tier, only its lints". A new row in `test_integration_test.go` asserts exit 1 and no stand-in run; the existing tierless row asserts the new notice. `gatecli/doc.go`, `gateslot/doc.go` and the `Config.Gate` comment say the call runs the lints.
- F3: step 3 of the fragment sends a no-test rerun through the same gate call with `-p:<property>=true` after `--`, when the tier's command holds `{args}`, and never the raw command. Otherwise, or if the rerun still runs no test, it names the tests to the parent.
- F4: two rows in `scopes_integration_test.go`: the prime (no parent branch) over an inert-only edit runs whole, and a pair whose origin names a vanished parent branch runs whole and prints the stays-whole line.
- F5: the `testCmd` help in `cli.go` says scope all runs every tier whole unless every changed file is inert, which runs no tier.
- F6: `template.yaml` describes scope all with the inert condition, and says that while tiers is empty every key runs with nothing to run, and that once a tier is declared each tier a key names must be declared.
- F7: the fragment header, `darnengine/doc.go`, the `GateFixed` and `DarnTable` comments in `darnengine/spec.go` and the `DarnFragmentMakeChangeGated` comment in `contracts/stencils/stencils.go` state the writer's single measuring run of one test class.
- F8: `docs/dotnet-hub.md` says the Parent-Align and Finalize gates run no tier when the hub sets `inert_paths`, every changed file matches them, and no tier has `carry: false`.

Gate repair: the F7 comments in `internal/darnengine/doc.go` and `spec.go` wrapped a sentence across lines, which the comments lint flagged.
They now hold one sentence per line, with the exception stated as its own sentence.

Second gate repair: the F5 help text pushed the `lym gate test` Long text to 2091 bytes, past the 2000-byte cap `TestCLIForm_EveryCommand` enforces.
The scope-all clause now reads "or no tier when every changed file is inert", and two other help sentences are shortened.
`TestCLIForm` and `TestGofmt` in `cmd/lym` pass.

Verification: tier0 over `contracts/stencils`, `internal/darnengine`, `internal/gateslot` and `internal/gatecli` passed; tier1 of `internal/gatecli` (`TestGateTest*`) passed, the new rows included.

## Self-check

- fix-code-correctness, no findings.
- fix-consistency, LOW, `contracts/stencils/stencils.go:54` comment still said no gate: fixed.
- fix-consistency, LOW, `internal/darnengine/spec_test.go:110` case name "run no gate": no change; the name describes the assertion on the rendered step, which the exception does not alter. Disposition: fixed-differently by leaving the name, since it is accurate for the step's default rule and renaming would not change behavior.
- fix-affected-set, LOW, `contracts/stencils/stencils.go:54`: duplicate of the finding above, fixed.
- fix-affected-set, LOW, `internal/gateslot/doc.go:202` and `tiers.go:359` still said a tierless gate runs nothing: fixed.
- fix-proposal, MEDIUM, F3 sentence named a property the report line does not name: fixed (F3 above).
- fix-proposal, LOW, the measuring call's args reach every tier the key lists: fixed, a clause in step 3.
- fix-proposal, LOW, `docs/dotnet-hub.md` sentence unconditional: fixed.
- fix-proposal, LOW, `template.yaml` line 21 unconditional on `inert_paths`: fixed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
