---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:103
    summary: D6 and the acceptance criteria call `tmux` a tag a gate runs, but D6 Bound says no standing gate runs it until `test-tiers`
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:154
    summary: The test-isolation constraint says the check holds under every tag combination but does not say the tag-set list gains `tmux`, `llm` and the combined sets, or that exactly one TestMain must compile
---

### [MEDIUM] `tmux` is called a gate-run tag, but no standing gate runs it

**Location:** `decision-record.md` D6 last sentence (line 103), acceptance criterion on retagged tests (line 187).
**Issue:** D6 says "No test file may end up under a tag that neither a gate runs (`integration`, `tmux`) nor a compile check covers (`llm`)".
D6 Bound says the opposite for `tmux`: `cardGateCommand` stops running it, and only this task's own plan verify, or a hand run, covers it until `test-tiers` decides.
The criterion therefore holds for `tmux` only through one task's verify, and the sentence reads as if a standing gate exists.
**Fix:** Reword both places so `integration` is the gate-run tag and `tmux` is covered by this task's plan verify and by hand until `test-tiers`.
Say that the `go vet -tags tmux ./...` compile check also covers it, so the criterion stays checkable.

### [MEDIUM] The isolation constraint leaves the tag-set coverage and TestMain uniqueness unstated

**Location:** `decision-record.md` Constraints, test-isolation line (line 154).
**Issue:** The line says the `!integration` and `integration` TestMain pairs stay paired and "the check holds under every tag combination".
The existing check in `cmd/lyx/tmuxisolation_test.go` iterates only `isolationTagSets` (untagged and each tag alone) and asks that at least one TestMain compiles.
It never tests a combined set such as `integration,tmux`, and it does not test that exactly one TestMain compiles, which a duplicate would break.
The record does not say that the `!integration` files stay as they are and so compile under `-tags tmux` and `-tags llm` alone.
It also does not say that the tag-set list gains `tmux`, `llm` and the combined sets.
The plan writer cannot tell from "every tag combination" what the check must enumerate.
**Fix:** State in the constraint that the `!integration` files keep their constraint and are the TestMain under `tmux`-only and `llm`-only builds.
State that the check's tag sets cover each of `integration`, `tmux` and `llm` alone and the combined `integration,tmux,llm`.
State that exactly one TestMain must compile under each set.

### Settled focus questions

- Focus 1, shared helpers across tags: D6's last bullet requires each tag to compile alone, with shared helpers in an untagged file or a testkit kit.
  The acceptance criteria run `go vet` for `integration`, `tmux` and `llm` each on its own, so a combined vet passing while a single-tag build fails is caught.
  Not a defect.
- Focus 2, gate wording: D6 Bound and D10 agree with each other and with Out (`cardGateCommand` and gate selection belong to `test-tiers`).
  The only sentence that overclaims is the one in F1.
