```yaml
lens: fix-consistency
outcome: reported
```

Task: check, through the fix-consistency lens, that the round-2 change set agrees with its siblings, and find any change whose twin was left as it was.

Twins checked: the D1 Webster span against its criterion, Q4 and the `--amends-plan` bullet; the D1 subset rule against its Bound, its criterion and D6; the D2 cut and split against the Constraints, Q5 and the support-log Rejected alternatives; the D3 condition against Q6 and the Open risks; the D4 live-weft refusal against its Bound, its criteria, the family-project exemption and Q7; D5; the F11, F12 and F14 moves.

1. **MEDIUM.** Changed text: the new D4 bullet, "It is refused while the slug's weft branch is live by the test `fabricengine.Topology.Add` adopts by: a local branch, …".
   - Summary: the bullet contradicts D4's family-project exemption and the criterion "D4: `--record-file` on a family project whose raddle run has filed and keeps its worktree replaces the project's stored record."
   - Evidence: a raddle run that keeps its worktree has a standing pair, so its local weft branch exists. The project carries the run's slug, so `Add`'s test reads that weft as live (`internal/fabricengine/add.go`: "A pair is live when its weft branch exists locally"). The new refusal therefore refuses the very case the exemption ("exempt from the worktree refusal") and that criterion admit.
   - The exemption names only the worktree refusal, not the new weft refusal.

2. **LOW.** Changed text: the D1 rule "Each of its `### Added after Discussion` entries must equal, byte for byte, an entry of the current record, so it adds and rewrites none."
   - Summary: its twin in D6 was left stale. D6 still says the spec "adds that `lym loom decision replace` may", which grants the exception to "never edits or removes an added entry" as a whole.
   - Evidence: D6's first paragraph is unchanged. Under the new rule, replace may only remove an added entry, never edit one, so the spec sentence D6 dictates now overstates what replace may do.

3. **LOW.** Changed text: the D3 condition "or, on a run whose recipe has no Discussion approval row, when the run's board entry carries a stored record."
   - Summary: the support log's Q6 still answers "Yes, for every run whose entry carries a stored record (D3)."
   - Evidence: `support-log.md` Q6 is unchanged, while Q4, Q5 and Q7 were updated this round.

4. **LOW.** Changed text: the new D4 live-weft refusal.
   - Summary: Q7 was updated for the task-worktree wording but does not list the new live-weft refusal.
   - Evidence: Q7 now reads "refused while a run holds the entry or the slug has a task worktree in the hub, and refused for a run's task session".

5. **LOW.** Changed text: the D6 parent-directive bullet "on a record whose `## Goal` is a fenced board body, … the fold rewrites that body to carry the call".
   - Summary: the D6 review criterion was not extended to cover this new directive content.
   - Evidence: the criterion "Review: D6 — the discussion spec, the parent directive and the overview name `replace` and `--record-file`, when to use each rather than `add`, and the history reset, and the parent directive names the replace notice's opening" is unchanged.

Further tool calls: none. My context already held every file and code span this check needed.
