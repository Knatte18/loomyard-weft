```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency: checking that the round-1 change set agrees with its siblings, and that no change left its twin stale.

Changed spots checked against their siblings: Scope, D1 (`names`), D2 (`AbsoluteUserConfigDir`), D3 (exports), D5 (pull, collision, repair, bounds), D6 (sweep, recoveries, lift advice), D7, D8, the Slack constraint, the criteria, the Notes, and the support log's Round 1 entry, Rejected alternatives, Q6 and Q8.

1. **LOW.** Scope, the In bullet "A per-user config directory resolver shared by `internal/machinename` and `internal/githubclient`".
   The resolver is no longer shared. After the fix, `internal/machinename` resolves through `standalonestate.AbsoluteUserConfigDir` and `githubclient.CredentialDir` delegates to `standalonestate.UserConfigDir` (D2). The Scope bullet still names one shared resolver.

2. **LOW.** D2's heading "One per-user config directory resolver in `internal/standalonestate`" and its rationale "the directory has one resolver".
   The rewritten D2 adds a second exported resolver, `standalonestate.AbsoluteUserConfigDir`, beside `UserConfigDir`. The heading and rationale now contradict the decision body.

3. **LOW.** D3's export list (`FilePath`, `ReadFile`, `Names`, `CheckName`) and the sentence "`internal/machinecli` reads the file only through these and writes it through `internal/state` at the path `machinename.FilePath` gives".
   The writer also needs the lock path, which D1 sets as `machine.lock` beside the file, and none of the exports gives it. The plan writer must either have `internal/machinecli` name `machine.lock` itself or add an export the list doesn't have. That sits badly with the Notes' scan, which pins that no production file outside `internal/machinename` names the machine file.

4. **LOW.** Support log, Turn 3 lines 39 and 41: "advises a hub pause" and "D8 now isolates only `internal/machinename`'s own tests".
   The fix changed the twins of these lines (Rejected alternatives, Q6, Q8 and the new Round 1 entry) but left these two. A reader of Turn 3 now sees advice and a test scope that D6 and D8 no longer hold. This may be acceptable as history, since the Round 1 entry supersedes them.

I made no further tool calls.
