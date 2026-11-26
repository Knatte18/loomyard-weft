---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "The sidecar Warn says the next status-pane restore will replace the strand, not the proposed next start or resume: a resume that refuses or is a no-op runs no restore."
  - id: F4
    disposition: fixed-differently
    reason: "The proposed summary sentence says reed moves the view to the replacement's window, which holds only when the replace succeeds: the old window is killed before the new strand is added. The summary now says so, and resume's Long and file header, which said it never switches a tmux client, carry the same exception."
  - id: F5
    disposition: fixed
---

## Changes

- F1 (`b1297ce40`): `ensureStatusStrand` returns an error wrapping `errStatusStrandNotReplaced` on a failed replace.
  `start` logs it and carries on, and `resume` reports it in `status_pane`.
  The wrap also names the strand's GUID, which the removed Warn logged.
  `6e69c2f01` adds the missing test, after the gate's reach report: a smoke subtest where reed refuses the replace,
  and `resume` still spawns the Go driver and names the failure in `status_pane`.
  With the replace branch made to return nil, the subtest failed.
  The reach report still lists no mapped test for `ensureStatusStrand`:
  its tests run it in a `lym` subprocess, which the report says its map may not see.
- F3 (`b1297ce40`, `3d08948df`, `5a5c374ae`): the file header and doc comment of `sharedbootstrap.go` name `resume` as a caller, and the sidecar Warn is reworded.
  The last commit repairs the gate's two comment findings, an over-long header line and a wrapped doc-comment sentence.
- F2 (`45e21cf2e`, `dd1924e6f`): the smoke subtest for `resume` asserts that one live status strand replaces the stale one.
  With the restore closure in `wiring.go` made to return nil, that assertion failed.
- F5 (`7a5793433`): `resume`'s Short no longer says it starts nothing.
- F4 (`6bbf64907` and `_lym/landing/summary.md`, uncommitted by rule): the view-moving exception is stated in the summary, `resume`'s Long and its file header.
  The summary's Tests section also names the smoke test.

The `code-burler-fix` gate passed on the final file contents, which `6e69c2f01` then recorded.

## Self-check

- fix-code-correctness: no findings.
- fix-proposal, LOW (the sidecar Warn overstates what resume does): fixed with own change.
- fix-proposal, LOW (the summary's exception covers only a successful replace): fixed with own change.
- fix-consistency, LOW (the smoke comment states one fact twice): fixed as proposed.
- fix-affected-set, LOW (`resume.go` still said it never switches a tmux client): fixed with own change.

## Disputed

## Deferred

## Rulings
