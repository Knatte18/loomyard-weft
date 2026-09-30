# Card 11 — triage-docs

**Prosa:**
- `doc.go`
- `//docs/overview.md`
- `//contracts/specs/webster-spec.md`

**Intent:** Describe verify triage wherever the integration stage is documented.

In `internal/websterengine`'s package doc (`doc.go`), rewrite the "Integration-suite fork + in-process bisect + terminal escalation" section:
the fork captures output to a scratch log;
on `FAILED`, webster reruns the verify once, compares against the plan's starting commit, and classifies flaky, pre-existing, or regression;
the report gains `failures` and `triage`;
bisect localizes by the regressing identities;
a regression demotes `done` to `stuck`, while flaky and pre-existing keep the outcome and are recorded as warnings, a `summary.md` section, and a friction note.
Remove the sentence stating that a Master `done` over a FAILED suite fails loud.

In `docs/overview.md`'s webster row, name the triage step beside the integration-suite fork and SHA-bisect.

In `contracts/specs/webster-spec.md`'s summary-artifact section, add the `## Integration suite triage` section beside `## Integration suite failed`, and state that the failed section now also lists the regressing identities with their tails.

No module doc under `manifest/designs/` describes the integration stage, so none changes.
Use semantic line breaks in every changed paragraph and comment line.

**Commit:** 11: triage-docs
