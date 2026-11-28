```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review proposals the fixer carried out (fix-proposal lens).

1. **MEDIUM**: F1 changed routing (`orchroute.Destination`, walk.go; summary "Notice and ask routing"). A deliberately downed beam orch still takes its members' mail, indefinitely.
   - `orchcli.stopOrch` (stop.go) removes the reed strand but leaves `orchengine.State.Strand` recorded, so `Destination` still chooses that beam after `lym orch stop`, `lym beam down` (also `--all`), `BeamDown` from `battencli/cancelcmd.go:410` or the pawl Beam-Down row, or a beam taken over by another machine (orchcli/doc.go: "loses its orch strand and its processes on this machine").
   - Before, member asks, halts and the dobby's notices went up to a live orch or the prime. Now they queue for an orch that may never start here.
   - `ParentOrchRecorded` and `runParentRecorded` then report the run as having a watching parent. The flipped `battencli` row (prime unrecorded, member `m` → true) shows this.
   - The new wording "creel delivers it once the orch is live" (orchroute/doc.go, summary) says nothing about the case where it never is.
   - "Strand recorded" does not distinguish a probe misreading a running orch from an orch that was stopped on purpose. The fixer-report should at least state this trade-off for the parent.

2. **LOW**: F3 reworded `parent-directive-handback.md`. The review's example wording, "the run hands the stop back to its parent", still claims the stop reaches the named parent.
   - For a removed parent pair, `ParentOrchName` names `tst:po-removed:orch` (`parentof_integration_test.go`). The halt notice goes elsewhere, because `FromPair` falls back to the prime.
   - The same happens for a beam whose orch records no strand: the notice passes upward.
   - So the finding's premise holds in those two cases: the prompt says "Your run's parent is X ... hands the stop back to its parent", and X never receives it.

I found no breaking case for F2, F4–F8 or F9–F13 as carried out.

Tool calls (1 of 6):
- One grep over docs, contracts and internal for drift amend/defer caller wording and `BeamDown`/`orch stop` sites. My context did not hold the `BeamDown` call sites or any drift wording outside `driftcli`.
