---
verdict: APPROVED
findings:
  - id: F12
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:94
    carries: F11
    summary: "D4 step 2 reads the ready record in the wrong repository for a beam off this machine: `beam-ready.json` is committed on the beam's records (weft) branch and never pushed by `lym beam ready`, so `git -C <prime> show origin/<beam branch>:_lym/landing/beam-ready.json` finds no file on any beam and a flagged beam passes the check."
    fix: "In D4 step 2 (lines 93-96) replace the claim that the ready record is 'committed on the beam branch' and the remote read. State that `lym beam ready` commits the record in the beam's run-records worktree, so it sits on the beam's records branch `<beam branch>-weft`, with no push (`MarkReady` in `internal/beamcli/deps.go`, which calls `fabricengine.CommitAnchoredPaths`, which commits in `RecordsWorktree` with no push). Keep the read in the beam's worktree when it is on this machine. For the other case, read `git -C <records repo> fetch origin`, then `git -C <records repo> show origin/<beam branch>-weft:_lym/landing/beam-ready.json`, and add that an absent file on origin proves nothing for a beam on another machine, since the record may be committed there and not pushed: the orch then asks that beam's orch by `lym creel post --expect-reply`, or the operator, whether `lym beam ready` ran, and treats the beam as qualifying only on a 'no'. Change the D4 acceptance bullet (line 231) from 'the read of `_lym/landing/beam-ready.json` on the beam branch' to 'on the beam's records branch', and keep support-log Round 2's F11 line in step."
prior:
  - id: F8
    status: resolved
    answer: D2 now asks the live writer by `lym creel post --expect-reply` for the first three parts in its reply and writes no file; the orch writes the note in its own `.scratch/`. D3 takes a refusing writer, or a reply missing parts, as no live writer. A creel reply is not a file write, so no writer limit is crossed, and the Auto-mode line and the D2 and D3 acceptance bullets match.
  - id: F9
    status: resolved
    answer: "The fixer's reason holds. `judgeOrigin`'s pre-end errors (`internal/battencli/cancel.go:612`, `:620`) carry the same 'fix the cause it names, then run ... again' wording as the post-end ones, so only the way forward tells the cases apart, and a rerun is safe (`cancel.go:255`). The hub-pause refusal (`cancel.go:304`) names a hub-wide resume, the owned-elsewhere refusal (`cancel.go:715`) names an adopt, and the reached-parent, landed-elsewhere and nothing-left refusals name other steps, so all fall under 'every other error' and D7. D4 step 3 forbids a slugless `lym tender resume`, and the hub-pause refusal stops with the slug pause in place."
  - id: F10
    status: resolved
    answer: D4 step 3 confirms the code tag and the absence of the slug's `-weft` records branch on origin before step 4, bounds the wait at ten minutes with a manual tag push from the prime in the form `failedRemoteSteps` names, stops with the note's scratch path when the records branch stays listed, and D4 step 4, D6 and the owner Constraint name the tag fetch.
  - id: F11
    status: open
    answer: The read of the ready flag is now named and the refused-`project` outcome is stated, and the fixer's disposition on the second is accepted. The remote half of the read still misses the record, so F12 carries this finding.
---

## Findings

### [MEDIUM] D4 step 2 reads the ready record on the code branch, where it never is

**Location:** `_lym/discussion/decision-record.md:94`

**Issue:** Carries F11. The fix for F11 added a remote read, `git -C <prime> show origin/<beam branch>:_lym/landing/beam-ready.json`, on the premise that the record is 'committed on the beam branch'.
`lym beam ready` writes the record under the beam's `_lym/landing` and commits it through `fabricengine.CommitAnchoredPaths` (`internal/beamcli/deps.go`, `MarkReady`).
That function commits in `RecordsWorktree(l)` with no push (`internal/fabricengine/commitweftpaths.go:76-82`).
`_lym` is the records side (`pattern/PATTERN-lym-files.md`, 'Weft content is per-branch'), so the record sits on the records branch `<beam branch>-weft` in the records repository.
The code repository's beam branch holds no `_lym/landing/beam-ready.json`, so the read reports it absent for every beam, flagged or not.
For a beam whose worktree is on another machine the check therefore always passes, the cancel runs, and the moved member never starts (`contracts/orchrole/role-beams.md:61`).
The record may also be committed there and never pushed, so even the right branch on origin can read absent.
The local-worktree half of the read is correct.

## Rejected

- D4 step 3's rerun loop stops 'when a run again returns the same error', and an `end_only` envelope is no error, so a long-unreachable origin has no stated end to the reruns.
  Not raised: an incremental round raises only BLOCKING findings, and the orch's pause stays in place and a rerun is safe, so nothing is lost.
- D4 step 3's manual tag push has no stated outcome when the tag is still absent on the next check.
  Not raised for the same reason.
