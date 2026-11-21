Task: check the fix's code correctness through the fix-code-correctness lens.

no findings

The fix changed no code. It edited one markdown file, `_lym/landing/summary.md`. The Door line changed from `two-way` to `one-way`, and a sentence now says `runplan.VerifyGateReport` no longer writes the optional `triage` key of `verify-gate.yaml`. The finding's `fix` key said to leave the code unchanged. There is no function, branch, signature or test to run or break.

Tool calls made: none. My context already held the edited summary and the review, so nothing further was needed.
