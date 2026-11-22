```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: one finding.

**MEDIUM** — `harden-rubric-review.md` ("Carry the charter forward" bullet) and `RoundFacts` (`campaign_rows.go`, the new Charter line).
- **Summary:** the rubric and the facts now direct the judge to read the focus file and the decision record. The shared judge template still forbids that.
- **Evidence:** `bouncer-template-judge.md` line 4 says the judge "reads the facts file ... the latest review and the previous ledger, and never the artifacts". Line 22 says "you do not open them". Line 120 says "never open the artifacts or the fixer's diff".
- **Why it matters:** the rubric tells the judge to restate the charter's standing sections, and the new Charter fact names the file that holds them. The template, which is not touched, forbids opening the artifacts. The decision record is an artifact. The focus file may or may not count as one.
- **Effect:** a judge that follows the template may skip both files and lose the charter again, which is the F1 symptom.
- **Possible fix, if wanted:** state in the rubric that the facts' Charter and Operator's ask files are the one exception to "never open the artifacts". The review's constraint was to leave the shared template unchanged, so the rubric is where it can be said.

Everything else I checked agrees.
- **`harden-live-copies`:** no reference remains in code, yaml, docs or PATTERN.
- **`gatecli`:** the copies refusal at `internal/gatecli/run.go:451` fits the one-tier-on-`harden-live` rule in `CheckCampaign`.
- **Seed template and rubric:** the ban exception and the "never a test the charter bans" bullet agree.

Calls made:
1. `grep` of `bouncer-template-judge.md` for the facts, artifacts and focus rules, because my context lacked that template.
2. `grep` of `gatecli/run.go` for the copies refusals, to confirm the one-tier check matches the `CheckCampaign` change.
