---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/editdirective/editdirective_test.go:41
    summary: The new scratchBuildBan assertion pins the static prose of both edit-directive stencil copies, which PATTERN-test-speed forbids.
    fix: In internal/editdirective/editdirective_test.go, delete the scratchBuildBan const (lines 20-21) and its strings.Contains check with its t.Errorf (lines 41-43) from TestDirective_ReadsTheEnginesCopyWithoutBanner, and restore the test's doc comment line 16 to "its own tool wording is present, with no marker, leading comment or trailing newline." Keep the engine tool-wording, marker, leading-comment and trailing-newline assertions unchanged.
    origin: lens:code-tests
---

### [BLOCKING] Test pins the scratch ban's stencil wording

**Location:** internal/editdirective/editdirective_test.go:41

**Issue:** `pattern/PATTERN-test-speed.md:249-253` says no test pins a stencil's static prose;
a stencil test may pin only a marker set, a Fill round-trip, a Go-filled marker's position, a value or fragment Go fills, reads, names or chooses, or a rule across a stencil family, and review guards the wording.
`stencilkit.Seed` seeds the shipped registry, so the new `strings.Contains(got, scratchBuildBan)` check asserts the literal sentence of the shipped `edit-directive.md` copies.
`Directive` neither fills nor chooses that sentence; the existing `"Edit or Write"` and `"apply_patch"` rows pin the engine choice Go makes, and that is the only wording the test may hold.
The rubric's test-economy bullet also names a new assertion on a stencil's static prose as a finding.
Since `PATTERN-test-speed` is a listed constraint, the violation is BLOCKING.
The assistant reviewer reported the same defect; this finding merges it.

## Holistic review

The focus questions, each settled:

- Both engine copies carry the board entry's full ban: build, restore and compile, and git worktree, clone or checkout, for `.scratch`, `.lym` and `_lym`.
  With the Codex tool wording swapped for Claude's, the two copies differ only in their banner and tool-wording lines.
- The removed build-resolution guidance told agents how to make a build under `.scratch/` work, which the operator's decision forbids; dropping it is licensed by the board entry.
- The orch-role note states the same ban in the same terms and sits in `## Notes and orch rules`, beside the existing `.scratch/` rule an orch session reads.
- No other shipped instruction tells an agent to build or check out under these folders.
  `contracts/stencils/retro/retro-template-task.md:23` passes `--work .scratch/forkaudit`, which `tools/forkaudit/forkaudit.sh` uses for intermediate files only, not a build or checkout.
  Skills under `plugins/ly/skills/` and the other stencils use `.scratch/` for drafts.
- The landing summary's claims hold: both engine copies and the orch role carry the ban, and `internal/fabricengine/doc.go:655-657` still states the link's build-resolution fact, unchanged by the diff.
- The commits carry no `Pattern-Departure:` trailer and need none; no PATTERN change is declared.
