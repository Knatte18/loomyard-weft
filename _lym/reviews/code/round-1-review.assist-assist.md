---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lym/landing/summary.md:32
    summary: The two-way merge classification overlooks the persisted notice-state and setting behavior changes.
    fix: "Change the Merge danger door in _lym/landing/summary.md to one-way and explain that LandNoticeState drops member and beam.land_notice off now suppresses member notices; include the resulting loss of the description-update prompt in the blast radius."
    origin: handler
---

### [MEDIUM] Merge danger understates the behavior and record change

**Location:** _lym/landing/summary.md:32
**Issue:** The summary calls this a two-way door.
The diff removes the persisted `member` field from `LandNoticeState`
and changes an off member's landing from a notice with the description-update prompt to no notice.
The rubric explicitly rejects a two-way classification for a record-format or config-meaning change.
Tolerant decoding makes old records readable; it does not make the two versions' notice decisions equivalent.

Section 1 is implemented, including the follow-up on an allowed landed line.
The section 2 outcomes already exist at the review base:
F26 uses told code-worktree anchors; F2 uses the shared module constant; F27 has its keep marker;
F28 has no duplicate beam row; F5 uses a machine-local carry and reads folded run settings;
F13 preserves project legacy keys across saves.
No omission depends on assuming that every hub has discarded its retired fields.
The summary records `beam-pr-describes-itself` as separate work.

The four lenses found no implementation, affected-reader, coverage or scope defect.
The supplied current-tree gate reports show unique test coverage and killed mutants.
No build or test was rerun during this read-only review.

## Rejected

- Missing section 2 implementation: the requested fixes are already present at the review base.
- A member-specific JSON test gap: existing tolerant-state tests cover the unchanged decoding mechanism.
- Weakened follow-up assertions: the rows retain exact expected lines and now enable the setting that permits those lines.
  Removed membership-only cases exercise the retired exception.

