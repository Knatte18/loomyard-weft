# Review of PR #321 (refusal-audit), round 4: close the evidence holes

Rounds 1–3 hold, with reaching tests; build, vet and the integration suite pass.
Round 3's rule — a correctness halt clears only on evidence, never on an acknowledgement or a verb an agent can run — still has holes.
Fix the items below, each with a test reaching it from a realistic state, and update `contracts/specs/refusal-spec.md` and the package docs to match.
The two reviewer reports carry the full scenarios: `/home/knatte/Code/loomyard-LYXHUB/loomyard/.scratch/review-321-r4-gp.md` and `review-321-r4-fable.md`.

## A. Evidence must cover HEAD, not only the worktree

`AcceptPendingAudit` (auditledger.go:110-117) and the probes in gitwrap.go:234-247 compare the worktree against the last batch head.
Master can commit its suspect write through Bash; restoring the worktree file then passes `accept-audit` while HEAD still carries the commit, and Publish pushes it.
Require HEAD to equal the last recorded batch head (reuse `reconcileReportHead`) in `AcceptPendingAudit`, and HEAD to equal the run's start commit in the `--fresh` pending-drop check (runlevel.go:1101-1106).

## B. `recover-batch` must not clear findings it cannot check

`checkRecoveredSuspects` (suspect.go:197-249, called from recoverbatch.go:365; see recordbatch.go:252-264) skips findings with no path or a path it cannot check: a fabric reference, the scratch pause flag, a write into a sibling worktree.
Master's own failure ladder runs `recover-batch`, so such a batch goes done with no evidence, although `accept-audit` refuses the same findings.
A finding the recovery check cannot verify keeps the batch failed; its way forward is `lyx webster run --fresh`, as for `accept-audit`.

## C. A fork must not write webster's state

No `CheckFork` class (audit.go:147-209) covers a fork writing `_lyx/webster/state.json` or anything else under `WebsterDir` but its own report.
A fork that marks its own batch done is refused by `record-batch` as "already terminal" (recordbatch.go:155) before the audit at :186 runs, so the done checks and the fork audit never run and the batch lands.
Add a correctness class for fork writes under `WebsterDir` other than the fork's own report, and run the audit before the terminal-state refusal.

## D. Plan edits must not slip into the recorded hashes

- `lyx webster validate` (validate.go:240-246) and `record-batch` (recordbatch.go:484,527) re-record plan hashes without checking what changed, so editing `00-overview.md` and running `validate` skips every `rebaseline` refusal, and a fork that weakens its own card's `## verify:` through Bash is accepted.
  Both must refuse a changed plan file exactly as `rebaseline` does, naming `rebaseline --card` as the way forward.
- The `--fresh` pending-drop check excludes plan paths (runlevel.go:1095): check them too.
- The way forward named for a suspect plan path is `git checkout <head> -- <path>` (auditledger.go:121, runlevel.go:1023,1064, refusal-spec.md:83), which fails because `_lyx/plan` is not tracked on the task branch.
  Name a working restore for plan paths.

## E. Pick the evidence base by execution order

`lastBatchHead` and `runStartCommit` (suspect.go:23-50, rebaseline.go:106) pick by batch number, but `SequenceBatches` can run batch 02 before 01.
The named `git checkout <head> -- <path>` can then revert a later batch's committed work, and `--fresh` checks against a mid-run commit.
Pick by recorded execution order or git ancestry.

## F. `.lyx` writes outside webster's scratch

Master writes under `.lyx` outside `ScratchDir` read as ignored and so as policy (audit.go:336,371-377).
Add `filepath.Dir(geom.ScratchDir)` to `runDirs`.

## G. Smaller gaps in the same rule

- `checkRecoveredSuspects` (suspect.go:223-248) looks for the flagged blob only at the suspect path, so a `git mv` keeps it under a new name; look for the blob anywhere in the head tree (`git ls-tree -r <head>`).
- A base SHA that does not resolve (a cross-machine resume whose last batch commit was never pushed) makes `worktreePathDiffers` return a raw `exit 128` from `accept-audit` and `--fresh`; treat it as unverifiable and name a way forward.
- `refusal-spec.md:61` promises policy findings warn "after the card verify commands re-run and pass", but the run-exit path (runlevel.go:983-987) records them with no re-run: re-run there, or correct the row.
- Say in the spec, where the audit rows are defined, that writes are seen through Write/Edit/NotebookEdit only (shuttleengine/forkaudit.go:19,33), so a Bash write is caught only as a fabric reference: the audit catches slips, not an adversary.
