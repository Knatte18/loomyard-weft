# Fixer report — discussion, round 1

Every finding was validated against the code and fixed.
`lyx loom validate-discussion` passes on the result.

## Fixed

- **F1** (BLOCKING): confirmed that only `internal/loomcli/darnwiring.go` sets `VerifyMergeBase`, and that a loom run's round gate lints from the plan verify's last recorded pass (`internal/loomshed/verifygate.go`).
  D3 now gives the round gate's headroom lint its own base: a merge-base reader for the task branch's recorded parent branch, built as darn's is and told in every pair run.
  Telling the reader does not switch a loom run to the whole-diff form.
  A read error is a gate error, and a standalone run gets no reader and skips the lint.
  D1's rationale names where its premise is enforced.
  D3's rationale is rewritten, since the comment lint's merge-base scope exists only for darn.
  The round-gate acceptance line is restated against the merge base, and a line pins that the round-form comment lint base and impacted-set command are unchanged.
- **F2** (BLOCKING): with the merge base as the lint's base, a Publish merge-in moves the base onto main's content, so D2's never-blocked guarantee holds.
  An acceptance line covers a non-growing branch after a merge-in of a parent above the line, and a growing branch over it.
  D5's rationale is reconciled: Publish's failure record names the bytes over the cap, and the round gate's headroom finding demands the larger cut, to the line.
- **F3** (MEDIUM): D2's Bound names what the lint lets through: growth at or under the line, changes on main outside a gate, a merged tree above the line at Publish, and D3's bypasses.
  D3's Bound names a fork that skips its card gate, caught at the next round gate, and a standalone run.
  Both rest on the hard cap that every full verify runs.
  The no-usable-base pass is gone, since the lint no longer uses the round-form base.
- **F4** (MEDIUM): confirmed the raw `/proc/<pid>/stat` reads in `pairteardown`, `shedverbs` and `gatecli`.
  They are brought into scope: Scope's list, D8 (a third kit probe for the stat state letter; each package keeps its own fallback), the tier-purity allowlist rows for the two `integration`-tier files, the auto-mode assumption and the acceptance line.
- **F5** (MEDIUM): D3 now requires the round gate's and the verb's headroom finding to name the way forward.
  A Constraints line for `PATTERN-refusal-way-forward` adds the cause to the "Webster-Burler verify gate exhausted" row of `contracts/specs/refusal-spec.md` in the same commit, with a test that reaches the finding.
- **F6** (LOW): D4 requires an entry whose background file is created to gain its link, counted in the 15000-byte budget.
  `pattern.Check`'s `orphan-background` finding already fails an unlinked file, so the change states the requirement and its byte cost rather than closing a gap in enforcement.
- **F7** (LOW): the gofmt guard's acceptance line covers a source that does not parse.
- **F8** (NIT): D4's rationale says "about two entries".
  The auto-mode assumption now points at D4 instead of restating the room.
- **F9** (NIT): the six-file list is dropped from D7, and the snapshot moves into support-log.md's Q9.

The support log also gains two rejected alternatives for the round gate's base, and Q1 and Q7 are updated to match.

## Disputed

None.

## Deferred

None.
