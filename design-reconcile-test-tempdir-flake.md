# loomcli: TestLoomResumeReconcilesTheHubConfigBeforeArming flakes on TempDir cleanup: weft-bare/objects not empty

`reconcile-test-tempdir-flake` · Note · bug, loom, config

Imported from [issue #472](https://github.com/Knatte18/loomyard/issues/472).

## Seen

`webster-record-batch`'s Publish verify, 2026-10-08T19:14Z, after merging main at 5aa04c279 (which added this test with `binary-change-and-rules`):

```
--- FAIL: TestLoomResumeReconcilesTheHubConfigBeforeArming (4.22s)
    testing.go:1464: TempDir RemoveAll cleanup: unlinkat /tmp/TestLoomResumeReconcilesTheHubConfigBeforeArming2267336794/001/weft-bare/objects: directory not empty
```

The test body passed; only `t.TempDir` cleanup failed, so a process was still writing into the bare weft's object store after the test returned.
The likeliest writer is a detached `git gc --auto` (git daemonizes auto-gc by default) started by a push or commit in the hub reconcile, or the reconcile's hub-wide push, which is only logged on failure.
Main passed the same test at Finalize, so it is timing-dependent.
The log is `.lyx/verify/verify.log` in the `webster-record-batch` pair.

## Workaround

The orch resumed the run so Publish re-runs the verify.

## Expected

The test's fixture repositories never run auto-gc in the background (`gc.autoDetach=false` or `gc.auto=0` in the fixture config, ideally in the shared testkit hub fixture), and the reconcile waits for every git child it starts before returning, so `t.TempDir` cleanup never races a live writer.
