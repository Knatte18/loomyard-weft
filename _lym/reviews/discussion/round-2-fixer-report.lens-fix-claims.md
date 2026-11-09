```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD (fix-claims lens).

**[MEDIUM] Changed text:** D11 Bound, "`lym s` there runs `lym adopt --here` first, which takes the subject back, and that adopt's reconcile meets the tips through its local-commit stop."
**Summary:** The reconcile stops only on a branch that diverged or was rewound or rewritten. A tip that is only ahead of origin passes with no stop, so "meets the tips through its local-commit stop" is false whenever the new owner pushed nothing on that branch.
**Evidence:** `internal/fabricengine/originreconcile.go:145-146` says `BranchAhead` "means origin's head is an ancestor of the local tip". `:169-175` `Stops()` lists `BranchRewound, BranchRewritten, BranchDiverged, BranchGone, BranchDirty, BranchBusy, BranchLive, BranchAnchorStopped`, and `BranchAhead` is not among them. An ahead tip stays local as the re-adopting machine's own branch. Only a diverged one stops.

**[LOW] Changed text:** D11 new paragraph, "It pauses no run, since no dobby of the subject runs to start a pass."
**Summary:** `lym beam start` does owner-pause runs before this refusal. Its `prepare` runs `pauseOwnedRuns` on every `owned_elsewhere` entry before `beamUpRefusal` is reached, so "pauses no run" misdescribes the command the paragraph names.
**Evidence:** `internal/orchcli/start.go:484-486` calls `prepare` when `opts.Prepare` is set, before the refusal at `:498`. `:457` `paused, pauseWarnings := c.pauseOwnedRuns(reconciled)`. `:395-411` pauses each `EntryOwnedElsewhere` entry's pair.

**[LOW] Changed text:** the new D5 acceptance criterion, "an adopted beam's own-scope `owner` entry on its pair slug is turned into an `adopt` entry in place, which `lym tender resume <beam>` lifts and the run report names".
**Summary:** The run report builds its candidate slugs from the moved runs, the held/paused slugs and the front tasks. It names the beam's entry only if the converted beam slug is among the slugs the record lists as paused, and the D5 text the fix added does not say it is.
**Evidence:** `internal/adoptcli/runreport.go:65-67` and `:99-110` take candidates from `in.Before`, `in.Moved`, `in.Held` and the open front task. D5 says only "The record lists the slugs it paused", and the added sentence does not say the converted beam slug joins that list.

Claims that hold:
- `lym beam start` refuses a foreign owner before any dobby starts: `start.go:498` returns before `startDobbyStep` at `:509`.
- D3's "An adopted beam's own owner entry, which that step lifted": `adopt.go:399-408` starts `settled` from `adopted` and lifts each one.

Tool calls:
1. grep of `internal/orchcli/start.go` for the refusal and dobby-start sites, to check the beam-start order the fix claims; my context had only `start.go:370-459`.
2. grep of `internal/fabricengine` for the local-commit stop and `EntryOwnedElsewhere`, to check the D11 Bound reconcile claim; my context had no reconcile code.
3. `sed` of `start.go:478-512` and the `Stops()` grep in `originreconcile.go`, to see the refusal order and which branch states stop.
4. grep for `waiting_for_check`, to find the run report the D5 criterion claims names the entry.
5. `sed` of `internal/adoptcli/runreport.go:55-110`, to see which slugs the run report covers.
