```yaml
lens: fix-claims
outcome: reported
```

No findings.

Tool calls beyond my context:
- Grepped the `PushWithRetry` and `pushPlainWithRetry` callers, the `removeFields` callers with `runRemoveWithFlag`'s definition, and `PushCoalesced`'s head. My context had none of the callers outside the files I read, and that check bears on card 1's "every plain push keeps today's classification" and card 13's "only caller `runRemoveWithFlag`". Both hold. `removeFields`' only other caller is `TestRemoveFields` (`internal/fabriccli/removefields_test.go:127`), which is already on card 13's Edit list.
- Read `internal/loomcli/commitrecords_test.go` lines 1–62 and found the function enclosing `fabric.go:1151`. My context lacked the stub deps of `TestCommitRecordsVerb_Success`, which card 5's new row relies on. The stub `stubRecordsDeps` takes a `pushErr`, so the row is feasible. The function at `fabric.go:1151` is `runRemoveWithFlag`.
- Read `internal/gitrepo/push.go` lines 277–300 to confirm `PushCoalesced` reaches `PushWithRetry`. It does, through `pushWithRebaseRetry`, so card 1 is right to list `PushCoalesced` among the unchanged plain pushes.
