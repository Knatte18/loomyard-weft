```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: challenge the N1 and N2 proposals the fixer carried out as proposed.

- **MEDIUM — N2, `internal/gatecli/run.go` `flagShapeRefusal`:** the check treats any explicit target as "names a package", so `lym gate test --tier 2 ./...` passes.
  - `./...` is a target, so `len(request.targets) == 0` is false and the refusal never fires, yet that call starts every package's tmux tests. This is the whole-tier start the ruling forbids.
  - The refusal also passes `--tier 2 -- -run X` with no package. That call computes a diff scope and builds and starts a test binary for every package in scope.
  - The fixer chose "no target AND no filter" from the ruling's wording. The reviewer's proposal was exact, but a bounded package still leaves the hole. Requiring a target that isn't a wildcard, plus a filter, would close it.
  - Evidence: `request.runAgent && runsTmuxTier(request) && len(request.targets) == 0 && len(request.args) == 0`. The test table pins `package_target` and `filter_after_dashes` as allowed, and `./...` is not tested.
- **LOW — N2, `internal/gateslot/startedlines.go`:** `TmuxTier = 2` is hardcoded, as `BilledTier = 3` is.
  - A hub that declares its tmux tests in another tier number would have that tier unguarded and its tier 2 wrongly guarded.
  - The doc comment says the tier's tests "drive a real tmux server", which the gate does not verify.
  - PATTERN-test-speed fixes the numbering, so this is only a minor risk.
- **LOW — N2, uncovered paths:** `--gate` keys listing tier 2 are exempt, and the fixer documents that in `doc.go`. `lym gate card-verify` and `red-first` build their own `testRequest` with `runAgent` false, so they are also unguarded.
  - Evidence: `runAgent` is set only in the `testCmd` `RunE` literal.
- **LOW — N1, `contracts/orchrole/orch-role.md:107`:** the orch role stencil tells the orch to run `tmux capture-pane -p -t <pane>`.
  - If the orch session is launched through a shuttle spec without `AllowRawTmux`, the row now denies that instruction.
  - I did not verify whether the orch is a shuttle run spec. An opened session writes a creel-only settings document with no shell rows, so it is probably unaffected.
  - Evidence: grep hit at orch-role.md:107. The only other stencil with tmux commands is the sampler's, which the exemption covers.
- **No findings on the N1 mechanism.**
  - `AllowRawTmux` is a named spec field and is not derived from `Interactive`.
  - The Claude and Codex maps both read it.
  - The golden was regenerated with the field false.
  - Evidence: `settings.go` and `guards.go` map `SwitchDenyRawTmux` to `!allowRawTmux`.

Calls made:
1. Bash grep of `contracts` for tmux commands in stencils, to see whether any interactive-role stencil tells its agent to run raw tmux. The context lacked the stencil contents.
