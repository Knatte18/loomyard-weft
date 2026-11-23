---
verdict: CONTINUE
rationale: "Round 3 has no findings, but its live report is red (harden-live passed and started 0 test binaries), the operator's ruling bars approval with focus entries undriven, and the review's Coverage leaves the discard half of the killed teardown, the keep-local tag push, a failed step with no kill and adopt --archives over real owner tags undriven."
---

## Why not CONVERGED

The rubric rules CONVERGED only on a round whose review records no BLOCKING or MEDIUM finding, whose live gate is green, and whose reviewer model differs from the previous round's.

- Findings: none in round 3, so the decision rule's finding test passes.
- Live gate: red.
  The facts record `harden-live` as passed with 0 test binaries started, and `harden-live-3.json` names no failing test.
  A live run that starts no test binary proves nothing about the fixes, so the gate does not count as green.
  The fixer's two commits (`592c88c`, `7e498df`) added `--no-carry` to the hardener's live call; round 4's live gate must show that it starts the campaign's live tests.
- Reviewer model: round 3 ran `sonnet`; the round 3 focus file records round 2's as `fable`, so the models differ.
- Undriven focus: the review answers APPROVED, but its own Coverage lists focus entries it did not drive.
  The operator's delegate (`ly:orch`, creel #2408) rules that a hardener round never approves with its focus undriven, and asks for those entries as round 4's focus.
  That ruling is applied here.

## Focus departures

The review carries no `## Focus departures` section; nothing to ratify or reject.

## Prior items

The four prior answers (G1 to G4) map to the four round 2 keys that were open.
Each is recorded resolved in the ledger on a re-drive the review logs.
Two resolutions are partial and are re-driven in round 4 rather than treated as closed:

- G3 (`pending-teardown-rerun-skips-owed-archive`): only the records half was driven; the discard half was not.
- G1 (`archive-tag-plain-refs-tags-one-repo`): the relative URL spelling and the two-repository control were not driven.

No finding was relabelled.

## Not driven, and what the method did not show

- The discard half of the killed teardown.
- The keep-local tag drive (focus entry 4).
- A failed-step pending record without a kill (entry 5).
- `adopt --archives` over owner-archive tags written by the real `ArchiveOwnerTips` path.
- The two-repository control, the relative URL spelling, concurrent claims on a second hub and a sabotage proof of each fix.
- The call-by-call enumeration of `Fetch`, `Pull`, `UpstreamSHA`, `HasUnpushed` and `HasUnpulled`.
- The regions of the last focus entry (kills of `checkout`, `pull` and `reconcile`, hand-over and code-tip reset paths).
- Legacy plain `refs/tags/archive/` tags: no reader names them.
  The review recorded this as a rejected LOW because an incremental round raises only BLOCKING findings; round 4 asks the question as an entry so it is settled.
- The review's repro of focus 1 and 3 drew its proof of entering the mechanism from a hook log and a pending record, but no sabotage run.
