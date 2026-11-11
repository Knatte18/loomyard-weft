# Fixer report — discussion round 1

Target: `_lym/discussion/decision-record.md`. The support log is unchanged.

## Changes

- **F1 (BLOCKING), fixed differently.**
  D1's commit bullet now says the seam reports committed false when nothing is staged.
  `replace` then lands an empty commit carrying the same trailers through `fabricengine.CommitEmptyWithTrailers` (new), beside `commitEmptySnapshot`.
  The proposal had the seam wrap `gitrepo.Repo.CommitEmpty` and refuse on staged content; I did not do that.
  `CommitEmpty` refuses on staged content anywhere in the index (`internal/gitrepo/gitrepo.go:177-182`), and no step of `replace` can clear it, so the proposed way forward looped.
  This came out of the self-check.
  The seam now commits the record's path alone through a new `gitrepo.Repo` method that allows an empty commit and leaves other staged content staged.
  A plan-writer note asks Plan to confirm that git behaviour, since this round could not run git in a scratch repo.
  The D2 swept-bytes criterion is kept.
  A D1 criterion now requires a trailer commit at HEAD when HEAD already holds the bytes, including while another path is staged.
- **F2 (MEDIUM), fixed as proposed.**
  After a commit error, `replace` reads HEAD:
  - written bytes with this call's trailer: the call succeeds and reports `commit_error`;
  - written bytes without the trailer: it lands the trailer commit, and if that commit fails it refuses with re-run `replace`;
  - HEAD lacks the written bytes: it restores the prior bytes and refuses.

  New D1 criterion for the HEAD-already-holds case; the commit-failure criterion is reworded to "HEAD lacks the written bytes".
- **F3 (MEDIUM), fixed as proposed.**
  The lock bullet adds `writeCarryOver` in `internal/loomcli/wiring.go`, held across its write and commit, so the lock covers every Go writer of the record past approval.
  New D1 criterion for a carry-over write started during a replace.
- **F4 (MEDIUM), fixed as proposed.**
  The D2 trailer value carries the `--by` value, the record's anchor-relative path and the sha256.
  The read takes the newest trailer naming this record's path.
  A trailer whose sha256 matches no version leaves the history whole.
  New D2 criterion for another record's trailer.
- **F5 (MEDIUM), fixed as proposed (recommended option), widened in the self-check.**
  D3 refuses `add`, `replace` and `store` on a raddle run whose filing is done, in every mode, as an extension of `perchFilingRefusal`.
  The way forward names the filed part, or `--record-file` on the family's project.
  D1's admission bullet points at it.
  New D3 criterion.
- **F6 (MEDIUM), fixed differently.**
  - Kept from the proposal: the replace notice's opening `Parent decision record replaced with lym loom decision replace` (new) is in D1, and D6 has the parent directive treat a LOOM line with that opening as the parent's instruction.
  - Not done: holding a Merriam strand back and re-entering Webster.
    The self-check showed a fresh Master prompt renders only the record's `Added after Discussion` entries (`internal/websterengine/runlevel.go:734`), so a re-entered Merriam does not carry the replaced record.
    Holding back a running run's Merriam would also be a regression from `add`, which reaches it inline.
  - Done instead: D1 refuses `replace` on a run standing at one of its Webster rows, so no strand that reads no record is live during a replace.
    The way forward names `add`, then a later `replace` once the run has left those rows.
  - Criteria: the D1 criteria now cover the notice opening and the Webster refusal, and the D6 review criterion names the opening.
- **F7 (MEDIUM), fixed as proposed (drop option).**
  D4 drops the live-batten clause and reads no batten liveness.
  The evidence: `internal/boardcli` imports nothing from batten, and liveness is read only from this machine's records (`registeredBattenAlive`, `liveBattenPID`).
  The D4 Bound states that a batten start after the check, and a batten run on another machine, are not seen.
- **F8 (MEDIUM), fixed as proposed.**
  D4 refuses an entry whose slug has a task worktree in the hub.
  The way forward names `replace` in that worktree or `lym batten cancel <slug>` first.
  The self-check added an exemption for a filed family project, whose run reads no record again.
  The D4 refusal criterion was updated, and a new criterion covers the exemption.
- **F9 (MEDIUM), fixed as proposed.**
  D4 validates the payload before the record file is written, and a failed save writes the former record file back, as `FileRaddle` does.
  New D4 criterion.
- **F10 (LOW), fixed differently.**
  The research answer, with its evidence, is recorded in D4:
  - batten's finish and cancel tear the pair down with `Remote: true` (`internal/battencli/wire.go:447`, `internal/battencli/cancel.go:717`);
  - `Topology.Remove` tags, pushes and deletes the weft branch;
  - `fabricengine.Topology.Add` adopts only a live branch.

  The branch therefore stays live only while the pair stands or after a failed remote step.
  At first I made `seedDiscussedRecord`'s commit carry the trailer.
  I took that out after the self-check showed it hides Open risk 3, where a stale re-seed would no longer be flagged.
  The adopted-branch case is now an Open risk, with its way out: a `replace` with the record's own bytes.
- **F11 (LOW), fixed as proposed.**
  Carry-over blocks must equal the current record's, byte for byte, with none added or dropped.
  The refusal criterion now covers an added block.
- **F12 (LOW), fixed as proposed.**
  The IdentityText sentence was reworded.
- **F13 (NIT), fixed as proposed.**
  The sampler strand with `--by operator` was added to the caller list and to the caller criterion.
- **F14 (NIT), fixed as proposed.**
  Notes for the plan writer, second bullet, deleted.

## Self-check

- fix-claims, MEDIUM, a fresh Merriam session does not carry the replaced record: fixed differently (Webster-row refusal in place of the hold-back; see F6).
- fix-claims, MEDIUM, "the only liveness read" is false: fixed as proposed (both liveness reads named).
- fix-claims, LOW, the teardown tags the local tip, not origin's: fixed as proposed.
- fix-consistency, MEDIUM, `store` still admitted on a filed run: fixed as proposed (D3 refuses `store` too).
- fix-consistency, MEDIUM, the seed trailer silences Open risk 3: fixed by removing the seed trailer (see F10).
- fix-consistency, LOW, D2 heading, D2 Bound and D6 still name the newest replace only: resolved by the same removal, since only a replace cuts again.
- fix-consistency, LOW, the `board` value has no caller `--by`: resolved by the same removal.
- fix-consistency, LOW, Open risk 1 against the Merriam hold-back: resolved, since the hold-back is gone.
- fix-consistency, LOW, the "neither" wording of the commit-failure criterion: fixed as proposed (now "HEAD lacks the written bytes").
- fix-way-forward, MEDIUM, a staged-index refusal loops: fixed differently (a path-limited empty commit in place of `CommitEmpty`; see F1). Merged with the fix-proposal MEDIUM.
- fix-way-forward, MEDIUM, the D4 worktree refusal leads into the D3 refusal for a filed family project: fixed differently (worktree exemption for a filed family project, and the D3 way forward names `--record-file` on the project).
- fix-way-forward, MEDIUM, re-entering Webster names no verb on a running run: fixed differently (see F6). Merged with the fix-proposal MEDIUM.
- fix-way-forward, LOW, the fallback trailer commit's failure is unstated: fixed as proposed (refuses with re-run `replace`).
- fix-proposal, MEDIUM, the F1 staged-index loop: merged into the fix-way-forward finding above.
- fix-proposal, MEDIUM, the F6 Merriam hold-back on a running run: merged into the fix-way-forward finding above.
- fix-proposal, LOW, the F5 "any row" refusal covers a family filing's repair: fixed differently. The refusal stands, since no split reads a call recorded during a repair. The way forward now also names `--record-file` on the project for a family-wide call.

Every fork returned a report.

## Gate repair

- `citation-symbol-missing` on D4: `fabricengine.Add` did not resolve, since `Add` is a method of `Topology`. The citation now reads `fabricengine.Topology.Add`.

## Disputed

None.

## Deferred

None.

## Rulings

None.
