```yaml
lens: fix-proposal
outcome: reported
```

Findings for the fix-proposal lens. I checked F1, F2, F3, F4 and F6, which were fixed as proposed, and left out F5, the decision finding. I found no case against F3 or F6.

1. **MEDIUM — card 9's `TestCaptureCurrent` row, "its `Sleep` releasing the held lock and returning at once, and the waits it was asked for are one second, then two"**
   The row as stated cannot pass.
   If `Sleep` frees the lock on its first call, the next poll finds it free, so `Sleep` is asked for one wait only: one second.
   The assertion "one second, then two" needs a `Sleep` that frees the lock only on its second call.
   The proposal in review F2 has the same contradiction.
   The wording should say the fake frees the lock on its second call, or assert a single one-second wait.
   Evidence: card 9's Intent says it "polls the lock, waiting through `Sleep` from one second, doubling…". `awaitCaptureEnd` in `internal/driftengine/wait.go:18-28` checks the lock and then sleeps, and the current capture is described as doing the same.

2. **LOW — card 15, "A weft path is judged by content: … no change when it was already uncommitted at the snapshot and its content now equals the hash or copy the snapshot recorded"**
   The rule does not say what happens to an uncommitted deletion.
   Case: a weft file under the reviews directory is deleted but not committed at the snapshot. Loom's status commit later stages that deletion, and `WeftChangedSince` names the path.
   No content hash or copy was recorded for a missing file. Unless "absent equals absent" counts as equal content, the row is Stuck with no edit by the judge, which is the case review F4 set out to close.
   Evidence: `statusCommitPathspec` (`internal/loomcli/wiring.go:102-115`) stages the whole reviews and loom durable directories, deletions included. Card 15's snapshot records "a content hash of every changed tracked path and a copy … of every untracked path", and says nothing about a deleted one.

3. **LOW — card 24, "its callers in `TestDriftBaseline`, `TestDriftAwait` and `TestDriftVerify` drop the argument"**
   Today `verify_test.go` calls `seedScopeRecord` four times in `TestDriftVerify` and once in `newVerifyFixture`.
   Card 16 rewrites both on `commitStore`, so by card 24 `TestDriftVerify` may no longer call it. Card 24 would then list a member with no change and state a step there is nothing to carry out.
   If the rewrite does keep the call in `newVerifyFixture`, card 24 lists `TestDriftVerify` but not `newVerifyFixture`, the member that holds it.
   Evidence: today's callers by function are 16 in `TestDriftAwait`, 4 in `TestDriftVerify` and 1 in `newVerifyFixture`. Card 16 lists `newVerifyFixture` under Edit and describes "the rewritten `verifyFixture` over `commitStore`".

On F3: `driftengine` already keeps its own row constants (`verifyRow`, `captureRowBaseline`) beside `loomshed`'s `NameDrift*` and does not import `loomshed`, so the new `expectRow` follows existing practice.

Calls made:
- One Bash call over `internal/driftengine/await_test.go`, `internal/driftengine/verify_test.go` and `internal/loomshed/loomshed.go`. It listed which functions call `seedScopeRecord` and the row-name constants in `loomshed`. My context had only the per-file call counts and none of `loomshed`.
