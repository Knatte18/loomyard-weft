```yaml
lens: fix-consistency
outcome: reported
```

Task: check that every sibling decision, criterion and cross-reference the fix touched still agrees with the changed text (fix-consistency lens).

Findings:

1. **MEDIUM.** D1's `PushOwed` bullet ("renamed from `PushOwedOffline` … Its refusal names the owed push and its last error, which `runfsm` carries in its inputs and verdict") and D5's new `PastParentMerge` bullet ("also sets `PastParentMerge` when the run's local parent branch holds its parent-side merge ahead of its upstream").
   - Summary: both changes rework landed `runfsm` code, and Scope's Out list still says a decision "fills or calls them and never redesigns them".
   - Evidence: Scope Out names `internal/runfsm`'s states, act rule and `runfsm.Judge` as landed and not to be redesigned. `internal/runfsm/act.go:25-30` defines `PushOwedOffline` as "owed because origin is unreachable" and `PastParentMerge` as "origin's fetched parent branch holds the task's work". The `ReasonPushOwed` refusal text is at `act.go:115`. The fix renames one input, rewrites a refusal, adds a field to the inputs and the verdict, and widens another input's meaning, but leaves the Out sentence unchanged.

2. **MEDIUM.** D6 step 1's new sentence: "A hold this machine wrote stays as it stands … the commit writes only the owner and the registry entry, and the adopt goes on from step 2."
   - Summary: `--take-hold` refuses a hold that names other subjects, but this own-hold path has no such check.
   - Evidence: D6 says `--take-hold` "refuses a hold that names other subjects, naming the hold's subjects". The new own-hold path keeps any hold this machine wrote whatever its subjects. So `lym adopt B` under this machine's own hold for `A` runs B's check, and step 8's green check "clears the hold" that covers `A`.

3. **LOW.** D5's upgrade notice now says a run whose entry became a pause intent needs `lym tender resume <slug>` after the green check.
   - Summary: D6 step 8 ("the runs go on by the act rule") was not qualified to match.
   - Evidence: the D6 criterion "the task's green re-check clears the hold and the runs go on with no hand lift" was not qualified either. Both read as if every run resumes. The runs the upgrade paused stay `paused` (`act.go:97`).

4. **LOW.** D5's upgrade bullet: "When the first read finds an open adopt fix task …, the hold is `adopt-fix`."
   - Summary: the sentence just before it, and the D5 upgrade criterion, still say the first read "writes one Bolt `adopt` hold" without noting the `adopt-fix` case.
   - Evidence: in D5, "When at least one entry became a pause intent … the first read writes one Bolt `adopt` hold naming this machine". The criterion adds the `adopt-fix` case only as a later clause after "writes one Bolt `adopt` hold".

5. **LOW.** D10's launcher bullet: "when any dobby of the hub starts on this machine for a worktree this machine owns".
   - Summary: the closing line "Beams need no launcher change." now sits beside a rule under which a beam's dobby start writes the hub's launcher, so the line reads as contradicting it.
   - Evidence: D10's bullet ends "Beams need no launcher change." directly after the changed sentence.

Tool calls:
- grep of `tendercli`, `tenderengine` and `fabricengine` for the pending-flag payment: checked whether a pause lift pays a code-side flag on its own, because my context lacked how the lift pays deferred pushes. The result was inconclusive, and I raised no finding from it.
- grep of `tendercli/deps.go` for deferral: same reason.
- grep of the record and the support log for leftover `hold`, `PushOwedOffline`, "prime dobby", "not yet read" and "only the operator": looked for twins of the renamed terms left behind, because my context does not show the whole changed file. Only the support log's historical lines remain.
