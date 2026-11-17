```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F10 fix change set for places where a twin of a changed text or behaviour was left as it was (fix-consistency lens).

**Findings**

1. **LOW**: `internal/proc/doc.go:56`, "The call reports that kill, and after any kill at the cap it kills what is left in the scope".
   - After the F1 change, `treeKilled` is the scope's `oom_kill` count alone.
   - The scope is still ended after an outside SIGKILL of the command, which sets `killed`. So the "after any kill at the cap" wording is slightly wider than what `treeKilled` reports.
   - `proc/doc.go:68`, "It reports whether the kernel killed any process of the tree at the cap", still holds.
   - The `memorycap.go` comment on `RunMemoryCapped` ("After any kill at the cap, every process left in the scope is killed too") has the same gap. It is minor.

2. **LOW**: `internal/gatecli/redundant.go:108` and `internal/gatecli/testmap.go:97`, "A test binary past memory_quit_mb is killed and named on stderr."
   - Both help `Long` texts were left unchanged.
   - F2 now makes a guard-killed run "not judged" in `lym gate redundant`, and "failed" with a logged reason in `lym gate testmap`.
   - `internal/gatecli/doc.go` and `docs/overview.md` were updated to say so.
   - The help pages are the only twin left stale. Skipping them is plausible given the earlier trim below the `Long` cap, but it leaves a visible mismatch.

**Checked, consistent**

- Every `RunAlone` caller that has a guard sets `GuardKilled`: `redundancy.go`, `redundantmutants.go` and `testmappackage.go`. The only caller left without it is `cmd/testtiming/redundancy.go`, which starts no guard.
- Every other `guardOwnTests` caller was updated to the two-value return.
- `redundantmutants.go` treats a `GuardKilled` run as no kill, skips its rerun, and does not set `FailsUnmutated`.
- `template.yaml`, `docs/overview.md` and the `doc.go` Memory guard section agree on the redundancy report step.
- The `memoryguard.go` file header is unchanged and still accurate.

**Calls made**
1. `sed proc/doc.go 50-75`, `grep MemoryCapped doc.go`, `grep memory guard testmap.go/redundant.go`: my context lacked `proc/doc.go` and the verbs' help text.
