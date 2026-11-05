Lens fix-affected-set: I reran each set-premised finding's search and checked whether the fix covers every member.

**Findings**

1. **MEDIUM** — F1: the gate's twin was left as it was.
   - Changed text: `NewDriftGate` in `internal/driftengine/check.go:370`, now `clearable := report.Skip == "" && report.BaselineFault == ""`.
   - `internal/driftcli/cli.go:167` still has `clearable := report.Skip == "" && report.BaselineFault == "" && report.RecordsFault == "" && report.SealFault == ""`.
   - It then sets `failing := clearable && (CaptureFault != "" || len(unexpected) > 0)`, which feeds `would_pass`.
   - `lym drift check` therefore still reports `would_pass: true` for an unexpected cell or failed capture beside a records or seal fault, while the real gate now fails. The two disagree.
   - Fix: narrow `clearable` in `cli.go` the same way, and add a row to the matching `driftcli` test.

2. **LOW** — F4: the new constant duplicates an existing one.
   - Changed text: `recordsMismatchHalt` (new, `expect.go`).
   - `verify.go:32` already defines `recordsMismatchReason` with nearly the same text and the same use, `haltForParent(...)` (`verify.go:120-121`).
   - Two constants now carry the same stop reason in one package.
   - Fix: reuse `recordsMismatchReason` in `expect.go`, or share one constant.
   - The `verify.go` text names `lym drift amend <slug> --restore` and `lym loom resume`, so it satisfies the test fragments the fixer added.

3. **LOW** — F4: another row still reaches the same refused way forward.
   - Changed text: `amendRows` now admits Drift-Expect only.
   - `await.go:99` (Drift-Await) calls `WriteRecords`, and so would hit the plain error naming `--restore`.
   - `Amend` refuses at Drift-Await, and a test row pins that refusal ("ARowBeforeDriftExpect" uses Drift-Await).
   - `baseline.go:139` is not affected: it writes only when no scope is recorded, and every records commit writes `scope.json`, so a mismatch there always has a committed scope and no write.
   - The review named only Drift-Expect and Drift-Baseline, so this is a set member the review missed.
   - Fix: halt for the parent at Await on a mismatch and admit it in `amendRows`, or record it in the fixer-report as left open.

**Checked, no gap**
- F5: `ExpectSession` is read in only two places, `expect.go:135` and `expect.go:222-223`, and both were changed. The judge mark (`JudgeSession`) is a separate field and untouched.
- F3: `SubexpIndex(keyGroup)` is used only at `diff.go:146`. `repeatedKeys` and `writeListing` already treat an empty `Key` as unkeyed, so the fix agrees with them.
- F6: no other test or doc names `TestDeferWaive`.
- F4 docs: the amend admission sentences in `doc.go:155`, `amend.go:61` and the `amendRows` comment all name Drift-Expect now. `verbs.go:37` and `doc.go:153` describe Defer, not Amend, so they are correct as they stand.

**Calls made**
1. Bash, one grep batch: `RecordsFault`/`SealFault`/`WriteRecords`/`Mismatch`/`ExpectSession` sites. The context lacked sites outside the files the fixer read.
2. Bash, one grep batch: `keyGroup`, `waive` and the amend-row doc sentences. The context lacked the other files that use the key pattern and the test-name search.
3. Bash, one sed batch: `cli.go:150-185` and `await.go:60-110`. The context lacked both files, and findings 1 and 3 rest on them.
