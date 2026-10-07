# Round 2 fixer report

Verdict was APPROVED with five findings; all are fixed in `decision-record.md` or `support-log.md`.
Round 1's fixes were checked after the review was saved and none regressed.

## Changed

- **F1 (LOW):** D4's Bound no longer says the refusal precedes the run directory.
  It now says the refusal happens before any artifact is written and before reed creates a pane, and that it blocks a new run, one adopting a session by `Spec.ResumeSessionID` included.
- **F2 (LOW):** D4 now states that an empty value is refused whatever its source, a zero-value `shuttleengine.Config{}` included, and that test fixtures handing `Prepare` a `Config` set both TTL fields from the template defaults.
- **F3 (LOW):** the seam acceptance criterion now names the vocabulary half as a review check under `PATTERN-shuttle-provider-seam` and keeps the existing test only as the import-seam check.
- **F4 (NIT):** the `PATTERN-spawn-observability` citation is dropped from D5 and from Constraints; the Constraints entry now gives the real reason, that the resolved value exists only in `claudeengine`.
- **F5 (NIT):** the support log's nested-key rejection now states the reason that actually separates the designs, and notes the roles map needs the open-map declaration either way.

## Deferred

None.
