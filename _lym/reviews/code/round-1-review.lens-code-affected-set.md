```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change the diff makes, find everything outside the changed lines that calls, reads, asserts or describes it, and report what the diff leaves wrong.

**Per change: the affected set, and whether the diff covers it**

1. **`mergeCauses` head-mismatch text changes from "the beam's head" to "the expected head"** (`internal/pullrequest/mergestate.go:111`).
   - `TestReadMergeState` `HeadMismatchNamesBothHeads` (`mergestate_test.go:75`): updated. Covered.
   - Callers that print `Causes`: `internal/beamcli/pr.go:350` (pr status) and `internal/beamcli/land.go:310` (ready / mark-ready). No beamcli test asserts the old text, and searches for "not the beam's head" find no test or fixture. Covered.
   - `internal/beamcli/land.go:331` has its own "not the beam's head" message in a beam-only context. That text is not `mergeCauses` output. Not affected.
   - `contracts/`, `PATTERN.md` and `pattern/` hold no copy of the text. Covered.
2. **`ReadMergeState` parameter `beamHead` renamed to `wantHead`, and the `MergeState.MergeableNow` doc changed.**
   - Callers pass the argument by position (`beamcli/pr.go:350`, `beamcli/land.go:310`, `landingshed/finalize_pr.go:503`). No change needed.
   - `pullrequest/doc.go:38-40` names no parameter. Covered.
   - The test-local constant `beamHead` in `mergestate_test.go:38` is a fixture name, not the renamed symbol. No old parameter name is left in production code. Covered.
3. **`mergeOnGitHub` reads the merge state on a `RefusalOther` and retries while the pull request can merge.**
   - `landingshed/doc.go:638-643`: updated. Covered.
   - Doc comment of `mergeOnGitHub` (`finalize_pr.go:458-460`): updated. Covered.
   - `prMergeRetryWindow` doc (`finalize_pr.go:21`) still says only a refusal "as pending" is retried. Left stale (finding 2).
   - `RefusalOther` doc (`pullrequest/client.go:75-76`) still says "none of which a retry clears". Left stale (finding 1).
   - `pullrequest/doc.go:44` is neutral. Covered.
   - `finalize.go:215-235` makes no claim about retries. Covered.
   - `contracts/` stencils and specs, `PATTERN.md` and `pattern/` describe no merge-refusal behaviour. Covered.
   - PATTERN-github-auth: the read goes through `landing.api`, the existing pullrequest API. Fits.
   - PATTERN-landing: still inside Finalize under the lock. Fits.
4. **`mergeRefusalReason` gains a `state` parameter, the "GitHub reports:" suffix and the mergeable-now resume-only text.**
   - Sole caller `mergeOnGitHub`: updated.
   - Tests `AnotherRefusal…` and `ABeamsOtherRefusal…`: updated.
   - Its own doc comment (`finalize_pr.go:571`) and `landingshed/doc.go:633` still say every refused merge of a beam's pull request names `lym beam pr status <slug>`. Left stale (finding 3).
5. **Commit messages.**
   - The `Root-Cause:` trailer on 9f602d61 names the defect: `mergeOnGitHub` halted on every `RefusalOther`, including the transient "not mergeable". The diff changes exactly that branch.
   - No commit carries a `Pattern-Departure:` or `Loosened-Check:` trailer.
   - In the two changed existing test rows, assertions were added and none were widened or removed.
   - Covered.
6. **Declared PATTERN changes.** The directive declares none. Nothing to check.

**Findings**

1. **LOW, consistency.** `internal/pullrequest/client.go:75-76`
   - **Summary:** The `RefusalOther` doc says it covers refusals "none of which a retry clears". The diff exists because GitHub's "Pull Request is not mergeable" is classified `RefusalOther` and clears on retry once mergeability is computed, and Finalize now retries it. The contract the diff relies on is left false.
   - **Evidence:** `client.go:75-76`; `finalize_pr.go:500-509`.
2. **LOW, consistency.** `internal/landingshed/finalize_pr.go:21`
   - **Summary:** The `prMergeRetryWindow` doc says it is how long a merge refused "as pending" is retried. The diff also retries a `RefusalOther` within this window while the merge state reads `MergeableNow`.
   - **Evidence:** `finalize_pr.go:21`; `finalize_pr.go:498-512`.
3. **LOW, consistency.** `internal/landingshed/finalize_pr.go:571`, and `internal/landingshed/doc.go:633`
   - **Summary:** Both docs still say a beam's refusal halt names GitHub's message, the pull request's status verb (`lym beam pr status <slug>`) and the resume. The new early return in `mergeRefusalReason` for a state that can merge runs before the beam branch, so a beam's halt in that case names only `lym loom resume` and never the status verb.
   - **Evidence:** `finalize_pr.go:577-580` (the early return comes before the `BeamRecord` check at :583); `doc.go:633`.

Tool calls:
1. Searched the code, contracts and specs for the old texts ("not the beam's head", "second person approves", "none of which a retry clears", `RefusalOther`), because my context lacked the repo-wide set of readers for the changed messages and docs.
2. Listed the `contracts/` docs that mention Finalize, grepped stencils, and read `finalize.go:215-235` and the head/cause lines of `beamcli/doc.go`, because my context lacked the other docs that describe Finalize's merge and the beam's merge-state reading.
3. Searched `contracts`, `PATTERN.md` and `pattern/` for mergeability and merge-refusal wording, because my context lacked any rule document or stencil that might describe the changed halts.
