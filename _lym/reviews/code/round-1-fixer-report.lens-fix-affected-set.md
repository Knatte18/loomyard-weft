```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind the set-premised findings (F2, F4, F1) and check that the fix covers every member.

Findings left unchecked: none of the set-premised ones. F3, F5, F6 and F7 name single sites rather than sets.

1. **MEDIUM**: `_lym/landing/summary.md`, the caller list and `**Blast radius:** fabric, shed`.
   - **Summary:** the text says that outside fabric it is `shedtransient.Class` that changes, but a direct caller outside both fabric and shed also changes.
   - **Evidence:** `internal/battencli/wire.go:1180` calls `fabricengine.AddOptions{...}.RetryOriginReadInTransit("beam", ...)`, so it now backs off on a missing origin.
   - **Evidence:** `shedtransient.Class` or `Mark` also drives `internal/battencli/land.go:107`, `internal/battencli/arm.go:975`, `internal/beamcli/pr.go:492` and `internal/loomcli/arm.go:604`, among others.
   - **Change:** name the batten call in the list, and widen the blast radius to cover batten and the other `Class` users, or say that they follow.

2. **LOW**: `_lym/landing/summary.md`, "Every caller that classifies through this function follows".
   - **Summary:** the list omits `rejectionRetryKind`, whose verdict flips for this error.
   - **Evidence:** `internal/fabricengine/pushclassified.go:357-370` returns `(true, false)` for `IsTransportFailure`. A missing path previously fell to `default` and returned `(false, true)`, which is retryable. The caller is at `pushclassified.go:341`.

F4: `unmounted` appears only in `doc.go` and `transport.go` among tracked Go and markdown files outside `_lym`, and both are updated.

F1: the missing-origin test fixtures I saw in the search all use absolute paths, so the bare-name rule does not reach them. That search was a filtered sample capped at 20 lines.

Tool calls (3 of 6):
- A grep for `unmounted`, `IsTransportFailure` and `RetryOriginReadInTransit`: to rerun the F2 and F4 searches over the changed tree.
- A grep for `rejectionRetryKind`, `shedtransient.Class`/`Mark` and `TransientGitTransport`: my context did not show the indirect consumers.
- A grep for test remotes pointed at missing paths and for git's message strings: my context did not show test fixtures outside the diff.
