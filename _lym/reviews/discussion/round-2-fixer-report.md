# Fixer report — discussion round 2

Every finding's premise was checked against the code it cites and holds.
`lym loom validate-discussion` passes after the fixes and after the self-check edits.

## Changes

- F1 (BLOCKING), fixed as proposed: D2 splits the read at the fabric boundary.
  `internal/fabricengine` gains two general seams: first-parent commits carrying a trailer key with its value, and an anchored file's history with commit SHAs.
  `internal/loomcli` owns the `Lym-Record-Replaced` key, parses its value and makes the cut in `recordHistory`.
  The `PATTERN-fabric-boundary` constraint line is restated to match.
- F2 (MEDIUM), fixed as proposed, by the refusal option: D4 refuses `--record-file` while the slug's weft branch is live by `Topology.Add`'s test, applied at the moment of the upsert so an in-flight teardown counts.
  The way forward names waiting for the teardown or following its failed remote step's way forward.
  Open risks bullet 4 is dropped, since the refusal closes the case.
  The D4 Bound and refusal criterion list the new refusal, and the support log records the rejected Record-Write re-seed.
- F3 (MEDIUM), fixed as proposed: D3's stored-record condition applies only to a run whose recipe has no Discussion approval row, and `store` matches it.
  A new D3 criterion covers `add` on a run re-entered into its Discussion segment.
- F4 (MEDIUM), fixed as proposed: D1 refuses replace from past the plan's approval through the recipe's last Webster row.
  The D1 Webster criterion covers the span before Webster, and the `--amends-plan` bullet says the fold comes once the run has left its Webster rows.
  Q4 in the support log matches.
- F5 (MEDIUM), fixed as proposed: D2 cuts at the newest version matching the trailer's sha256 and committed at or before the trailer commit.
  A D2 criterion covers the X, Y, restored-X sweep case. Q5 matches.
- F6 (MEDIUM), fixed as proposed, by the rewrite-the-body option: D6 says a fold on a record whose `## Goal` is a fenced board body rewrites that body.
  D1 names that replace permits rewriting that body.
- F7 (LOW), fixed as proposed: D3's filed test also counts a run whose entry is no longer a task, or whose filing the board holds by `boardengine.Board.HoldsRaddleFiling`.
  A D3 criterion covers a family filing whose store write failed.
- F8 (LOW), fixed as proposed, by the subset option: D1 refuses a file whose added entries are not each byte-equal to an entry of the current record.
  The Bound states replace cannot rewrite or insert an added entry, and the IdentityText sentence names a dropped PATTERN bullet as reviewed text.
  The D1 refusal criterion covers a rewritten or inserted added entry.
- F9 (LOW), fixed as proposed: D2's Bound says the trailer guards against a mistaken writer, not a forged commit.
  D4's Bound says the run-agent refusal guards against a mistaken call, not a forged or unset strand name.
  The support log's content-marker rejection no longer claims only a lym verb writes a trailer.
- F10 (LOW), fixed as proposed: Open risks states the run report counts only the added entries a record still holds.
- F11 (LOW), fixed as proposed: the `PATTERN-refusal-way-forward` line names D1 and D3, and says D4's unbound boardcli refusals name their way forward too.
- F12 (LOW), fixed as proposed: the plan-writer note is deleted, and D1's empty-commit bullet states the git 2.53.0 fact.
- F13 (LOW), fixed as proposed: Q7 and the Turn 3 bound-breaker 5 line name the task-worktree refusal.
- F14 (LOW), fixed as proposed: the `gitrepo.Repo.CommitEmpty` rejection, the batten-liveness explanation and the teardown mechanics move to the support log's Rejected alternatives.
  D4 keeps "It reads no batten liveness", the run guards line and the live-weft rule.
- F15 (NIT), fixed as proposed: D5's Bound states the mixed-binary limit on `record_body_sha256`.

## Self-check

- fix-way-forward, BLOCKING; fix-consistency, MEDIUM; fix-proposal, MEDIUM (merged): the live-weft refusal refused a filed family project whose run keeps its worktree, against D4's exemption and criterion.
  Fixed as proposed: the exemption covers the live-weft refusal too.
- fix-claims, MEDIUM; fix-way-forward, LOW (merged): `lym fabric remove <slug>` does not clear an origin-only live weft.
  Fixed with my own change: the way forward names the failed remote step's own way forward instead of a fabric command.
- fix-claims, LOW; fix-proposal, LOW (merged): `HoldsRaddleFiling` needs part slugs D3 did not name.
  Fixed as proposed: D3 names the run's raddle proposal's part slugs.
- fix-way-forward, MEDIUM: the crash-window refusal's ways forward looped.
  Fixed as proposed: that refusal names `lym loom resume` on the run first, which finishes the filing and writes the store marker.
- fix-way-forward, LOW: the D1 subset refusal had no way forward. Fixed as proposed.
- fix-consistency, LOW: D6's spec sentence let replace edit added entries. Fixed as proposed: replace may remove, never edit, an added entry.
- fix-consistency, LOW: Q6 was stale against D3. Fixed as proposed.
- fix-consistency, LOW: Q7 lacked the live-weft refusal. Fixed as proposed.
- fix-consistency, LOW: the D6 review criterion did not cover the fenced-body fold. Fixed as proposed.

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
