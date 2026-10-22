---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: PATTERN.md:1
    summary: the merged PATTERN.md is 16084 bytes, 416 over the parent tip and past the headroom line, so the decision record's no-growth rule breaks and the headroom lint fails at landing
    fix: Cut at least 416 bytes from other entry lines of PATTERN.md so it is at most 15668 bytes, the size at the parent tip fac869bb3, without changing any rule a line states. Re-apply the parent's shortened wording of the lines the merge reverted to the pre-parent text (see `git diff fac869bb3 HEAD -- PATTERN.md` for the lines, among them the cwd-resolution, told-geometry, sparse-hub-base, fabric-vocabulary and hub-containment entries) and keep the task's shortened PATTERN-owner-machine line, whose full declared text already lives in pattern/PATTERN-owner-machine.md. Check the result with `lyx loom lint-pattern`, which runs patternlint.Lint against the merge base.
    origin: lens:code-pattern-fit
  - id: F2
    severity: LOW
    class: consistency
    location: internal/loomcli/ownercheck_test.go:25
    summary: no test pins the order of the parent's pause refusal and the task's owned_elsewhere refusal in lyx loom start and lyx loom resume, so a run owned elsewhere with its owner pause standing could swap its first way forward from lyx resume to lyx adopt unseen
    fix: Add one row to the table in internal/loomcli/ownercheck_test.go for a run whose slug a live pause entry labelled pauserecord.OwnerCaller covers and whose owner verdict is elsewhere, for both start and resume, asserting the refusal is the pause's one naming `lyx resume` and that the owner verdict seam is not called. Reuse the existing refuseCovered fixture of the same package if one exists.
    origin: lens:code-tests
---

### [BLOCKING] Merged PATTERN.md grows past the headroom line

**Location:** `PATTERN.md:1`
**Issue:** Both sides cut other entries' wording in the same overview lines, the parent to make room for its own additions and the task to pay for its `PATTERN-owner-machine` line.
Git merged the overlapping edits without a conflict, and most of the net cut was lost.
The file is 15703 bytes at the merge base, 15668 at the parent tip, 15699 at the task tip and 16084 in the merged tree (`git show <rev>:PATTERN.md | wc -c`).
`pattern.HeadroomBytes` is 15564 (`internal/pattern/check.go:24`), and `pattern.Headroom` (`internal/pattern/headroom.go:11`) fails when the head is longer than both the base and the line.
The loom verify gate reads the merge base through `headroomBase` (`internal/loomshed/verifygate.go:88-112`), which after this merge-in is the parent tip, so the landing fails with "The branch grew PATTERN.md past its headroom line, so no test ran" and a 416-byte cut.
The decision record's "Added after Discussion" decision says PATTERN.md must not grow, so the task upholds neither it nor the parent's smaller overview.
`git diff fac869bb3 HEAD -- PATTERN.md` shows the merge restored the pre-parent wording on most entry lines, while the parent's additions (`PATTERN-pause-holds-prompts`, the `PATTERN-verified-tree` hold wording, the `PATTERN-review-round` verdict wording) survive.
The `PATTERN-owner-machine` line is under `pattern.MaxEntryLineChars` (360) and its declared text is verbatim in `pattern/PATTERN-owner-machine.md`, so only the total is wrong.

### [LOW] Order of the pause refusal and the owner refusal is unpinned

**Location:** `internal/loomcli/ownercheck_test.go:25`
**Issue:** The parent's `PATTERN-pause-holds-prompts` has `lyx loom resume` refuse with the pause's way forward, and the task's `refuseOwnedElsewhere` runs after `refuseCovered` in `internal/loomcli/start.go:505-510` and `internal/loomcli/resume.go:262-270`.
A run owned elsewhere with its owner pause standing therefore gets `lyx resume`; after the lift, the next call gets the `owned_elsewhere` refusal naming `lyx adopt`.
The decision record (D6) intends this chain, and the merged code follows it.
No test combines a covering pause with an owner verdict, so swapping the two calls breaks nothing.

## Rejected

- Fork finding (code-tests): no test combines the owner pause with the parent's session claim in `adoptcli`.
  Adopt calls `deps.Lift` and never reads the `ResumeReport`, `internal/adoptcli/adopt_test.go:305-320` covers the standing-entry wording, and the parent's `internal/tenderengine/resume_test.go` pins the `Moved` claim path, so the interaction has no uncovered behavior of its own.
- Fork finding (code-affected-set): the same PATTERN.md growth, filed MEDIUM.
  Merged into F1 and kept at BLOCKING, since the decision record states the rule and the headroom lint fails the landing.
- Fork finding (code-affected-set): the same ordering point as F2 as a separate wording finding.
  Merged into F2, since the behavior is intended and only its test is missing.
