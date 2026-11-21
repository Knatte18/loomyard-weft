# Finalize waits out a mergeability GitHub has not yet computed

Finalize halted on `Pull Request is not mergeable` when GitHub had not yet computed the pull request's mergeability.
The pull request read `MERGEABLE` a minute later, but the halt named a second approval or a required check.
Finalize now reads the pull request's merge state after such a refusal, waits out an uncomputed mergeability, and retries the merge.
It halts only on a definite refusal, and the halt names the cause GitHub reports.

Closes #161

## What changes

In `internal/landingshed`, `mergeOnGitHub` handled only a pending refusal as retryable.
A refusal of kind `RefusalOther` now has `pullrequest.ReadMergeState` read the pull request.
That reading already waits out an unknown `mergeable_state`, so Finalize sleeps and retries the merge while the reading says the pull request can merge.
Retries stop once `prMergeRetryWindow` (one minute, shared with the pending-refusal retry) has passed.

The halt reason now depends on the reading:

- A refusal that outlasts the window while GitHub reports the pull request can merge names `lym loom resume` alone.
- A reading whose only cause is an uncomputed mergeability names GitHub's message and that cause.
  Its way forward is to wait and run `lym loom resume`, with no approval or `lym batten cancel`.
  A beam's halt also names `lym beam pr status <slug>`.
- A reading that reports other causes appends them to GitHub's message as `GitHub reports: ...`, and keeps the existing way forward.
- A failed read of the merge state halts through `githubFailure` and never merges a second time.

In `internal/pullrequest`:

- `MergeState.MergeabilityUncomputed` reports a state whose only cause is a mergeability GitHub has not computed.
- `ReadMergeState` and `mergeCauses` name their head parameter `wantHead`, no longer `beamHead`, since Finalize now passes the task head.
  The head-mismatch cause reads `head X is not the expected head Y`.
- The `RefusalOther` doc now says it covers a "not mergeable" refusal raised while mergeability is still computed.

## Design decisions

- GitHub's merge response gives no separate signal for an uncomputed mergeability, so the refusal's own message cannot decide.
  The merge-state reading decides instead.
- The retry window bounds when a retry is scheduled, not the whole wait.
  Each merge-state reading can wait up to a further window, and a scheduled retry sleeps its backoff first.
  The worst case is a halt within about two and a half minutes, stated in the `landingshed` doc comments.
- A method refusal and a pending refusal keep their existing handling.

## To check by hand

- The merge-state read failure, in `internal/landingshed/finalize_pr_test.go`.
  `AFailedMergeStateReadAfterANotMergeableRefusalHaltsAndMergesNoSecondTime` covers a read that fails on its first status call;
  check that it covers the failure paths a reviewer cares about, such as a failure on the required-checks call or on the sleep.
- The halt advice for an uncomputed mergeability, in `mergeRefusalReason` in `internal/landingshed/finalize_pr.go`.
  Check that "wait until GitHub computes whether the pull request can merge, then run `lym loom resume`" is the way forward you want
  when the window passes with mergeability still uncomputed.
- Nothing was run against live GitHub; the tests script the GitHub calls.

## Merge danger

**Door:** two-way
**Blast radius:** landingshed
