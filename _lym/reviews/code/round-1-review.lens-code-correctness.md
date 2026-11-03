```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the drift diff (cases run per function or branch).

**Per function or branch, case run, verdict**
- `passWhenSwitchedOff`, config load error: Stuck "…; then lym loom resume". Correct.
- `passWhenSwitchedOff`, `Enabled=false`: Done, and the producers return before any spawn or commit. Correct.
- `passWhenSwitchedOff`, `Enabled=true`: falls through. Correct. In all three producers the call sits after `requireSeams`, so a nil `Config` still fails loudly.
- `passWhenSwitchedOff` while a capture child runs, switch flips off: Baseline returns Done and leaves the child running. It then writes its record unobserved. Acceptable and edge-only, so no finding.
- `captureAgain`, non-breakaway start error (child never records its start): Stuck with `CaptureHaltPrefix` and a way forward naming `lym drift defer`. `Defer`'s `atCaptureHalt` admits it, because it checks the prefix on `Status.Error` and a scope record exists by then. Correct.
- `Defer` at a capture halt, then resume: `AwaitedAt` is set, so Baseline takes no capture and Await is Done. Correct.
- `RecordScope`, agent at a running Drift-Baseline: record not found, so it is written. Found, so the "already recorded" refusal. Correct.
- `RecordScope`, parent at a blocked Baseline or Await: capture held is refused, `AwaitedAt` set is refused, otherwise it replaces. Correct for the single-step path (see F1 for the chain).
- `RecordScope` replacement, old `partial{a}` to new `full`: `dropCapture` is true and the capture is removed. Correct.
- `RecordScope` replacement, old `full` to `partial{a}`, or any replacement to `off`: capture kept. Correct.
- `StartCapture` with `AnchorDir`: the child's cwd equals the anchor, which `lymcwd.Resolve` requires. `driftcli`'s `anchor()` and the `loomcli` wiring pass `AnchorPath()`. Correct.
- `runStep`, relative dir joined on `AnchorDir`: correct.
- `startFailure`, with and without `FallbackDetach`: both messages are well formed, and the row adds the way forward in the no-fallback case. Correct.

**F1 — MEDIUM, design.** A scope replacement chain loses what the capture skipped.
- Location: `internal/driftengine/verbs.go:465-467`.
- Case run: a capture was taken under `partial{a}`. The parent replaces the scope with `off`, and the capture is kept. `cur.Skip` is overwritten with nil at line 466. The parent then replaces `off` with `full`, so `cur.Skip` is empty and `dropCapture` is false. A capture with no profile of `a` is now counted under `full`, and the compare on `a` fails or is missing.
- Cause: the drop test infers what the capture skipped from the current record's skip list. That list is not what the capture skipped once a replacement has overwritten it.
- Fix direction: keep the skip list the capture was taken under, for example a record field set at capture time. Alternatively, drop the capture whenever the replacement leaves `off` for a scope that compares anything the capture may lack.

**F2 — LOW, design.** A failed capture removal leaves the record inconsistent and unretryable.
- Location: `internal/driftengine/verbs.go:469-474`.
- Case run: `UpdateScope` succeeds and `os.Remove(p.Capture)` then fails with a non-ENOENT error. The error is returned, but the new scope is already stored.
- Result: a retry sees `cur.Skip` equal to the new skip list, so `dropCapture` is false and the stale capture stays counted. The old scope's capture is then compared under the new one.
- Fix direction: remove the capture inside the same locked update, or before writing the scope.

**F3 — LOW, design.** Gate-slot holder identity changed to the anchored directory.
- Location: `internal/driftengine/steprun.go:149`.
- `gateslot.Holder.Worktree` is documented as the worktree the holding run works in. Other holders (`internal/gatecli/run.go:584`, `internal/planglyph/typesload.go:126`) pass the worktree root.
- A drift step in an anchored hub now records a subdirectory there, so the holder listing (`internal/fabriccli/anchordeps.go:220`) shows a different identity for the same worktree. No consumer compares the value to the root. The hunk shows no need for the change, since the root is still available (`run.AnchorDir` is the only field).
- Fix direction: pass the worktree root to `stepRun` separately for the holder, or document `Holder.Worktree` as the working directory.

**F4 — LOW, design.** Every `StartCapture` error is now a capture halt.
- Location: `internal/driftengine/baseline.go:147-148`.
- Errors that are not "child did not start" also become "the baseline capture did not start … lym drift defer". Examples are failing to resolve the lym binary, failing to create the child log, and the capture-lock-held refusal.
- Offering defer for a resolve or log-file failure is arguably wrong guidance, since defer skips regression for an environment fault.
- Fix direction: map to the halt only the child-start errors (`startFailure`'s two kinds) and keep other errors returned.

**Tool calls (1 of 6)**
1. `grep -rn "\.Worktree\b" internal --include=*.go`: to find consumers of `Holder.Worktree` for F3, which my context lacked.
