---
verdict: APPROVED
rationale: "Reviewer approved; all six round-1 findings (F1 probe-error on unopenable pair, F2 MergeInProgress rationale, F3 batten verified, F4 driver_not_parked precedence, F5 one-repo conflict form, F6 Goal wording) are already reflected in decision-record.md and support-log.md, and the cited code anchors exist."
---

The round-1 report carries an `APPROVED` verdict with one MEDIUM, four LOW and one NIT finding, and no `## Focus departures` section, so there is no departure to ratify or reject.

I checked each finding against the current artifacts:

- **F1 (MEDIUM), half-broken-pair fallback.**
  Resolved.
  The probe decision now says a pair the probe cannot open, or whose record location cannot be resolved, is a probe error, never "not mid-merge".
  The probe-error acceptance criterion names the unopenable pair, and the support log's Rejected alternatives records the replaced rule.
- **F2 (LOW), `Fabric.MergeInProgress` rationale.**
  Resolved.
  The record now calls it "record-only" and states that the probe opens the pair from `l` itself.
  Interview item 5 matches.
- **F3 (LOW), batten treated as an assumption.**
  Resolved.
  The kind decision's rationale states that batten needs no change and why, the batten assumption and open risk are gone, and Q7 is resolved with evidence.
  `internal/battencli/wire.go:245` and `internal/battenshed/innerrun.go:293` confirm it.
- **F4 (LOW), precedence against `driver_not_parked`.**
  Resolved.
  The trigger decision covers the hand-back case, and a matching acceptance criterion exists.
- **F5 (LOW), form of the conflicted paths.**
  Resolved.
  The record specifies the one-repo form, and a matching acceptance criterion exists.
- **F6 (NIT), Goal wording.**
  Resolved.
  The sentence now begins "Today `start` checks no merge state at all".

The anchors in Notes for the plan writer exist: `mergeRecordExists`, `mergeBlocksMutation` and `foreignMergeStatePresent` in `internal/fabricengine/mergestate.go`, `MergeStateActive` in `mergestateactive.go`, and `mustSpawn` and the `StartNotParkedKind` refusal in `internal/loomcli/start.go`.

Under the rubric's leanness test I found no content to relocate or exclude.
The Open risks and Auto-mode assumptions carry operator-facing consequences that the plan writer needs, not deliberation narrative.
