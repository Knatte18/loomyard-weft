```yaml
lens: code-correctness
outcome: reported
```

The new branches mostly run correctly. I found one MEDIUM and one LOW finding, both in the halt text `mergeRefusalReason` builds when GitHub reports the pull request can merge.

**Each function and branch, the case I ran, and the result**

- **`mergeOnGitHub`, the merge succeeds the first time.** It breaks out of the loop with no merge-state read. Correct.
- **`mergeOnGitHub`, a failure that is not a refusal.** It goes to `githubFailure` as before. Correct.
- **`mergeOnGitHub`, a pending refusal within the window, then past it.** `retry` is true and `state` is nil. It sleeps and retries until the deadline, then halts through `mergeRefusalReason(..., nil)` with the old text. Correct, unchanged.
- **`mergeOnGitHub`, a method refusal.** `retry` is false, `state` is nil and it halts at once with the method text. Correct, unchanged.
- **`mergeOnGitHub`, a `RefusalOther` with `unknown` then `clean`.** `ReadMergeState` polls, gets `MergeableNow` true, the loop sleeps and merges again. Correct.
- **`mergeOnGitHub`, a `RefusalOther` with `blocked`.** `MergeableNow` is false, so it halts with `state` set and the causes appended. Correct.
- **`mergeOnGitHub`, a `RefusalOther` whose pull request head moved.** The message "Head branch was modified" reads as `RefusalOther`, and the reading adds the cause "head X is not the expected head Y". It halts naming the head. Correct.
- **`mergeOnGitHub`, a closed pull request at refusal time.** `ReadMergeState` does not poll a pull request that is not open. The cause is "closed", so it halts. Correct.
- **`mergeOnGitHub`, worst case for time (`unknown` for 59 s, then `clean`).**
  - First read ends near t=59 s, before the deadline. It sleeps 1 s, so the second merge is at t=60.
  - The second refusal's read can poll for another 60 s, to about t=120. Past the deadline, it halts.
  - The wait is bounded at about two windows plus backoff, which matches the summary's "at most about two minutes".
- **`mergeOnGitHub`, `MergeableNow` true but every merge refused as `RefusalOther`.** Each pass makes one quick read (`Status` and `RequiredChecks`). Waits are 1, 2, 4, 8, 16 and 32 s via `nextPRWait`, so the seventh refusal falls past the deadline and the loop halts. Bounded, with no growing reads. See the MEDIUM finding below for the halt text.
- **`ReadMergeState` fails with a cancelled context.** The sleep error comes back as is. `githubFailure` then goes to `transientFailure`, and `cancelErr` turns it into an error ending, not a halt. Correct.
- **`ReadMergeState` fails because the `Status` call fails.** The error is wrapped with `%w`, so the transient judgment still sees it. Correct.
- **`read, err :=` shadowing.** The inner `err` is scoped to the `if` block. The outer `err` from `Merge` is not used after it, so nothing is hidden. Correct.
- **`mergeRefusalReason` with `state` nil.** `reported` is "", so the texts are the same as on the merge base. Correct.
- **`mergeRefusalReason` with `blocked` and no checks.** The text reads `"...: <msg>; GitHub reports: blocked: a review or a required check is still owed; way forward: ..."`. Correct.
- **`mergeRefusalReason` with `state` set but `Causes` empty.** This cannot happen: `MergeableNow` is defined as `len(causes) == 0`, so that state takes the early return.
- **`mergeRefusalReason` with `MergeableNow`, on a beam.** The early return comes before the beam branch, so the text names only `lym loom resume` and not `lym beam pr status <slug>`. This contradicts its own doc comment; see the LOW finding.
- **`mergeRefusalReason` with `MergeableNow`, on a task.** The text names only the resume and drops the cancel. See the MEDIUM finding.
- **`mergeCauses` with a mismatched head.** The text is "head other is not the expected head beam1". Correct.

**Findings**

1. **MEDIUM**, class `design`, at `internal/landingshed/finalize_pr.go` in `mergeRefusalReason`, the `state.MergeableNow` early return.
   - Summary: when GitHub keeps refusing as `RefusalOther` for the whole window while its merge state reads `clean` (or `has_hooks` or `unstable`), the halt names `lym loom resume` alone.
   - Why it matters: the state reads `clean` and the merge is refused for a reason the merge state does not show, such as a rule or merge-queue requirement. That refusal outlasted every retry for a minute, so a resume repeats the same failing retry window. The way forward drops `lym batten cancel` and the "settle on GitHub" route, so the operator gets no exit the code itself offers.
   - Evidence: the new early return in `mergeRefusalReason`, `finalize_pr.go` around lines 577–581. The loop's retry condition, `retry = read.MergeableNow` with the deadline check, is at `finalize_pr.go` around lines 498–516.

2. **LOW**, class `consistency`, at `internal/landingshed/finalize_pr.go` in `mergeRefusalReason`.
   - Summary: for a beam, the `MergeableNow` early return leaves out `lym beam pr status <slug>`. This breaks the function's own doc contract, "A beam's refusal never names a cancel: it names GitHub's message, the pull request's status verb and the resume". It also breaks `landingshed/doc.go`, which says "A refused merge of a beam's pull request names GitHub's message, `lym beam pr status <slug>` and `lym loom resume`."
   - Evidence: `finalize_pr.go` around lines 569–582 (the doc comment and early return before the `BeamRecord != nil` branch) and `internal/landingshed/doc.go:634`.

**Tool calls**

- grep of `githubFailure` in `internal/landingshed`: the manifest did not show how a read error or cancelled sleep from `ReadMergeState` is judged.
- grep of `githubFailure` and `transientFailure` bodies (`ctx.go`, `finalize_pr.go`): to confirm a cancelled context becomes an error ending, not a halt.
