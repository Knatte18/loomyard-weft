---
verdict: CONTINUE
rationale: "F1 is a BLOCKING design finding: card 2 passes an external test unit such as internal/fabricengine_test unchanged to fabricvocab.SkipsBareCheck, so the check refuses what the scan admits; F2, F3 and F4 are LOW, LOW and NIT and not gating, with no relabels."
---

The latest round carries one BLOCKING finding, F1, class `design`, which makes the round gating under the decision rule.
F1 cites no `PATTERN` entry, so it gets no special weight from one, but its BLOCKING severity cannot be lowered.
F2 (`consistency`, LOW), F3 (`scope`, LOW) and F4 (`consistency`, NIT) are not gating.
No finding was relabeled and the review carries no `## Focus departures` section.
This is round 1, so there is no prior ledger and no recurring key.
