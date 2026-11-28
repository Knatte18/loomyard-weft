```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F10 fix in `internal/loomcli/prreview_mail_integration_test.go`.

no findings

What I traced, by reading only (I ran nothing):
- **Shared record path:** all subtests now use the pair's wired record path. They run in sequence, and each `WriteRecord` replaces the record whole.
- **Live record watch:** `pause` joins the sleep goroutine on both `select` branches, so `sleeps` and `now` stay synchronised. Each sleep advances `now` by at least `prWaitFloor`, so no extra `Sleep` call follows a change. A cancel dropped on the `changes` branch is picked up by the next wake's `cancelErr`.
- **Failed watch:** a watch that cannot open leaves a nil channel and the same wake count.
- **New commit:** the wired seams read at call time, so building `wired` before `CommitFile` is harmless. The expected head is the committed SHA.
- **`CommitsMissing`:** these paths never reach it, because the scripted status has an empty `HeadSHA`.
- **Callers:** no signature changed.

Extra tool calls: none.
