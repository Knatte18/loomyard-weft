---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md` D2 (the keep directive): settle whether the Decision states what the tool does with a directive that is not the comment line directly above a `func Test…` line, appears twice on one test, or sits on a test that is not a candidate, and whether a kept test stays in the covering pool the removable set is computed against."
  - "In `_lyx/discussion/decision-record.md` D3 (the narrowed-run splice): settle whether the Decision bounds a narrowed run whose `-tags` differ from the tag set named in the existing report header, a package whose listing or build fails in that run, and a missing `-out` file, and says exactly when a package section is removed."
  - "In `_lyx/discussion/decision-record.md` D1 (the untagged-file exclusion rule): settle whether the Bound covers a test that installs a git hook, alias or helper which makes git run module code out of process, and whether D4's hand check tells the reviewer to ask that question."
  - "In `_lyx/discussion/decision-record.md` Acceptance criteria: settle whether each bullet matches what D1 to D3 actually change, and whether the boundary behaviors the Decisions define have a scenario; judge only behaviors a Decision states, never call-site enumeration."
---

Focus targets the places where a guard or escape hatch is introduced and the attack-surface question applies.
None of these is on the rubric's `Do not flag` list, and none asks for content the discussion writer's stencil said not to gather.
