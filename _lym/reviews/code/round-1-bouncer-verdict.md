---
verdict: CONTINUE
rationale: "F1 is a design-class MEDIUM finding (the sibling census in internal/fabricengine/scratchlink.go ignores a sibling's real root .scratch), so the gating rule says CONTINUE; F2 to F8 are non-gating and no finding was relabeled."
---

Round 1 carries eight findings, none BLOCKING and six MEDIUM.
Only F1 has class `design`, and it is MEDIUM, so the loop continues.

F2 (consistency), F3 to F6 (scope) and F7 and F8 (LOW) are non-gating but are still fixed in the same pass, since class never changes whether a finding is fixed.
No finding's class or severity is departed from, and the review carries no `## Focus departures` section.
No PATTERN entry is cited by any finding, so none gets special weight.
