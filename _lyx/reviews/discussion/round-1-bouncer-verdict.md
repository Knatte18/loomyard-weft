---
verdict: APPROVED
rationale: "All nine round-1 findings (F1-F9) are fixed in the current record: D9 now carries a bounded exception for D7 and #325 §5 removals, D7 states its Bound, D3 bans gitkit/hubforge imports and splits scan from review discipline, D4 drops the websterengine tagging and names RepoFixture and the Tier 1 loss, the missing acceptance criteria exist, and D13's kept-unchanged narrative moved to support-log.md; no new rubric finding remains."
---

The round-1 review's verdict was `APPROVED`, with five MEDIUM, three LOW and one NIT findings.
A fixer then edited both artifacts, so this verdict judges the record as it stands now, not as the review saw it.
The review report has no `## Focus departures` section, so there is no departure to ratify or reject.

## Findings, checked against the current artifacts

- F1, D7/#325 §5 against D9: D9 now has a sole exception (`decision-record.md` D9), letting a D7 remedy-string assertion or a #325 §5 duplicate leave a protected test only when the owner test keeps asserting the behaviour and the removal is itemised in the test count.
  D7 gains a Bound naming the generic pass-through assertion, and Scope's #325 §5 bullet points at D9's exception.
  Resolved.
- F2, D3 spawn-import rule: D3 now bans `os/exec`, `internal/gitexec`, `internal/gitkit` and `internal/hubforge`, its Bound says Tier Purity covers only `_test.go` files (confirmed in `cmd/lyx/tierpurity_test.go`'s header), and the first acceptance criterion plants a `gitkit` import.
  Resolved.
- F3, D3 enforcement split: D3 names the scan-enforced rules (importer, `*cli`, spawn-import) and the review-discipline ones (no process beyond those imports, asserts nothing beyond setup and the `Call` check, `lyxbin` limited to `go build` of `./cmd/lyx`).
  The `lyxbin` Bound now says review holds it to that build.
  Resolved.
- F4, D4 and `websterengine/config_test.go`: D4 lists only the fabricengine files as untagged callers and says a comment-only mention is reworded and the file stays untagged; the file's only `gitkit.` text is the comment at line 24, as the review said.
  D4's Bound states that the tests using the moved helpers leave the default `go test ./...` run.
  Resolved.
- F5, Scope without criteria: new criteria cover `shedfake`'s `Call` helpers, the #324 §6 items, and #325 §6's reap-cycle table and `*lyxcwd.Location` builder.
  Resolved.
- F6, merged bullets: the Tier Purity/Hermetic Env and `fabricFixture` criteria are separate bullets.
  Resolved.
- F7, D13 relocation: D13 keeps its Decision, its "Changed by this task" list and its rationale, and states every other invariant is unchanged; the kept-unchanged reasons and part-2 notes are in `support-log.md` interview item 7.
  The completeness-before-leanness test holds: the constraints those reasons protected (Fabric Vocabulary, seam scans unchanged, Completion Signal rows) remain in `## Constraints` and the acceptance criteria.
  Resolved.
- F8, perishable narrative and tallies: D4/D5 rationale and the drift assumption are count-free.
  D4 keeps one clause, "today each keeps its own list, and the two disagree", which motivates the rule rather than narrating the lists; that is acceptable.
  Resolved.
- F9, `RepoFixture`: D4's Bound names `gitkit.RepoFixture` as banned until exempted by name; `internal/gitkit/gitkit.go` confirms it is a non-spawning exported type produced only by `CopyRepo`.
  Resolved.

## New issues

None.
Every new edge (the testkit reversal, the `lyxbin` exemption, `intentionallyNil`, the D9 exception, the Tier Purity tightening) states what it lets through and what bounds it.
Notes for the plan writer and the absence of rejected alternatives from the record are on the rubric's Do-not-flag list and were not weighed.
The support log has no Discussion-Review round entry yet; the stencil makes that the gate's to append, so it is not a finding against the writer.
