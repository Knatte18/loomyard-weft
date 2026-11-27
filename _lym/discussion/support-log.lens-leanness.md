```yaml
lens: leanness
outcome: reported
```

Task: a leanness review of the decision record and support log. It looks for content to move or cut, applying the completeness-before-leanness test first.

**Findings**

1. MEDIUM, `## Constraints`, the root-cause bullet and its sub-bullets. They repeat the `## Goal` fault list a second time and add the file paths. The cause is required content for a bug entry, so keep it, but cut the overlap: `## Goal` already names `resetUnnamedTip` and its file and the driver's 15 s/30 s/1 min waits. Those mechanism details belong only in Constraints. Evidence: Goal says "The Go driver re-steps … three times, after 15 s, 30 s and 1 min", and Constraints says "caps a network stop's re-steps at three waits summing under two minutes". The rule is said twice.

2. MEDIUM, `## Constraints`, the PATTERN bullets (pair-remote, run-fsm, landing, owner-machine, daemon-wakeups, config-layers, config-key-names, tolerant-records, gitrepo-client-boundary). Most of them restate what Decisions already commit to. Examples: D2's one-second floor duplicates the daemon-wakeups bullet, and D8 together with Scope "Out" duplicates the owner-machine bullet's fetch clause. Cut each bullet to a bare entry name and the decision it binds. Keep only the obligations no decision states: the test clock injection, tolerant ledger records, the key-name helper, and records-before-code for the new targets.

3. LOW, `## Notes for the plan writer`, bullet 1 (the PATTERN backgrounds needing edits). This duplicates the PATTERN-run-fsm Constraints bullet ("a background change the Webster card makes"). State it once, in the Notes. Drop the clause from Constraints.

4. LOW, D4 Rationale, "The ledger is per hub because a hub has one origin." This repeats the Auto-mode assumptions bullet and support log Q3. The assumption section is its home, so cut the sentence from D4.

5. LOW, D12. The two issue facts are deliberation that settles nothing for the plan beyond "name them in the change description". Keep them, because a plan writer who doesn't know they are settled would plan code for #218 and #254, so the completeness test holds. The "Rationale" line, though, only restates the entry's instruction. A one-line rationale is fine, so no cut is needed.

6. LOW, D6. Two sentences don't fit the decision. "`lym loom status` reads such a run as waiting … not as a failed halt" is behaviour. "The doc comment of … AnomalyTransientStop … say" is a doc fix whose acceptance bullet already exists. Keep it, since the review criterion depends on it. The prose is long but every sentence carries a requirement, so this is not a cut.

7. LOW, `## Open risks` bullet 1 (`fabric-core-push-origin` will carry D1–D7). This is a sequencing fact the board already shows (T23 depends on this task) and not a risk to this run. It duplicates the Scope "Out" bullet. Cut it.

8. LOW, support log `## Interview` Turn 1. Its exploration facts list largely mirrors the Constraints root causes. That is acceptable, because the support log is the place for narrative. No move is needed.

**What else I noticed (emphasis)**

- D5 bundles three mechanisms that would read better as separate decisions: the local landing push, the PR-landing wait on an owed parent push, and the Worktree-Teardown deferral. Each has its own acceptance bullet already, so this is structure, not leanness.

No further tool calls were made.
