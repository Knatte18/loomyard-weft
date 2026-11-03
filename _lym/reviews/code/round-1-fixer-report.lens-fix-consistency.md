```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the F1–F5 change set.

No findings.

- **Stale wording.** No remaining text says a replacement drops the capture when "the old scope skipped" a test. The `RecordScope` doc, `driftengine/doc.go` and the `lym drift scope` help in `driftcli/cli.go` all say "the capture skipped". `contracts/stencils/drift` and `docs/overview.md` hold no such sentence.
- **`StartCapture` start errors.** The `Call` doc comment, the `Deps.StartCapture` comment and `driftengine/doc.go` line 33 agree with the new split. A child that never recorded its start is a halt; other start errors return as before. The `driftcli` caller prints `err.Error()`, and `childNoRecordError` keeps the old message text, so `spawn_test` expectations hold.
- **`CaptureRecord.Skip`.** The only struct comparison, `stored != rec` in `capture_test.go`, was changed to `reflect.DeepEqual`. The `doc.go` record description was updated. `RunCapture` is the only writer that builds a fresh record; the later writes reuse `rec`, so they carry `Skip`.
- **`Holder.Worktree` doc.** The other readers (`gatecli/status.go:71`, `detachfront.go:920-925`, the `pool.go` log lines) only print the value. `gatecli/run.go:584` still passes the worktree root, which the new wording covers.

Tool calls (2 of 6):
1. `grep` for old-scope wording, `StartCapture` and `unstarted` across Go and Markdown. My context did not show which other files hold twin text.
2. `grep` for "replac", "capture halt" and `Holder`/`.Worktree` readers in docs, stencils and `gateslot`/`gatecli`. My context did not show those readers or the docs.
