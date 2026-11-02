```yaml
lens: code-correctness
outcome: reported
```

**Task:** correctness lens, running one concrete case through each function and branch the diff changes.

**Per-function list**

- **`Watch` after `advance(PhaseResuming)`, `err != nil`:** the `fail` call gets `r` from before the advance and `kept`. It lifts `r.Owned`, else the request file's `Owned`, else `kept`. Correct.
- **`Watch` after `advance`, `!ok`:** returns `r, nil` with no lift. A cancel lifts the entry itself, and a cancel during `hibernating` is refused, so the cancel case is covered. The leftover case is a request file that is gone or no longer ours. See F2.
- **`Watch`, `reconcileAtWake` error:** goes to `fail` with `kept`, which lifts, writes failed and sends one notice. Correct.
- **`Watch`, `target.Owned = kept` when `Owned` is nil:**
  - The lift uses `kept` as the identity, so the lift is by the right identity.
  - The notice text is wrong in this case. See F1.
- **`Watch`, `liftOwned` error:** `fail` lifts the same entry a second time, which is a retry. The returned `liftErr` is the second attempt's. The first error is kept only inside `reason`. The result is correct, and the entry is attempted twice, never lifted twice.
- **`Watch`, `finish(PhaseDone)` write error:**
  - The lift has already happened, and the notice is still sent.
  - `r` stays at `resuming`, with the error joined into the return.
  - The request file stays non-terminal after the daemon exits. That is acceptable, because `Submit` replaces a dead daemon's request and `Cancel` can end it.
- **`Watch`, request cancelled during the reconcile:** `finish` returns `(r, nil)` unchanged. The lift is by identity, so a second lift is a no-op. The notice still says "woke… lifted", as before. Correct.
- **`sendNotice`, boundary 1024 bytes and embedded `\n`:**
  - 1024 bytes is queued as is, and 1025 goes to the file.
  - `headline` is fixed text plus a fixed outcome in both callers, so the short line stays far under 1 KB and under creel's 8 KB.
  - The short line stays within the bound even if the file write fails. Correct.
- **`sendNotice`, retry loop:**
  - The waits run 1, 2, 4, 8, 16, then 30 until `waited >= 600s`.
  - Attempts stop with `waited` between 600 s and 630 s, matching the test's tolerance.
  - The last error is returned.
  - Every caller lifts before it calls `sendNotice`, and no lock is held, so the lift is never delayed.
  - A notice that never errors but is not posted counts as delivered. See F3.
- **`writeNoticeFile`:**
  - If `MkdirAll` or `WriteFile` fails, the notice goes to the log and the sentence says so. That is correct.
  - A second notice overwrites `notice.txt`, which is acceptable.
- **`fail`, long `reason`:** goes to the file, and the headline carries `outcome`. Correct.
- **`Cancel`, terminal request:**
  - `stopPID` is set only inside the closure, and the closure returns `errUnchanged` for a terminal request. So no kill happens and the request file is untouched.
  - `report.Cancelled` is false.
  - With `Owned == nil`, `liftOwned` returns nil, so `Lifted` is nil.
  - With `Owned` set and the entry already lifted, `Lifted` is non-nil with an empty `Resumed`. Before this change it was nil. See F4.
- **`Cancel`, request missing:** `readRequest` returns `found == false`, so it returns `report, nil`. Correct.
- **`reconcileAtWake`, `Stop != ""`:**
  - `stopsEntries` makes every entry `EntryStopped`, and `wakeReconcileFromReport` appends each to `Runs` (`internal/hibernatecli/reconcile.go:58-90`). So every stopped pair is held before the hub entry is lifted.
  - If `holdRuns` fails for one pair, that pair is named in `unheld` with `lym tender pause <slug>`. The hub entry is lifted anyway. This matches the existing non-stop behaviour.
  - With no runs, `held == ""`, so the notice is only `stop`. It still says "before the runs held here go on", although no run was held.
- **`reconcileAtWake`, other branches:** the skip, reconcile-error and unreachable branches return text only. The lift moved to the caller and is correct.

**Findings**

- **F1, LOW, design, `internal/hibernateengine/watch.go:142-153` with `internal/hibernateengine/wake.go:58`**
  - When `r.Owned == nil` and `kept != nil`, `reconcileAtWake` treats the live hub entry as someone else's. It skips the reconcile and the exemption, and its text says "the hub pause of X stands; lift it with `lym tender resume`".
  - `Watch` then lifts `kept`, and the headline says "its pause was lifted". The notice contradicts itself, and the reconcile never ran.
  - Evidence: `wake.go:58-59` and `watch.go:146-150`.
- **F2, LOW, design, `internal/hibernateengine/watch.go:138-140`**
  - `advance` returning `ok == false` returns without lifting `kept`. This happens if the request file vanished or is no longer this daemon's.
  - The daemon may still own a live entry in that case. A cancel is the only normal path to `!ok`, and a cancel lifts it. It is a gap only when the file is gone.
- **F3, LOW, design, `internal/hibernatecli/deps.go:113-115` with `internal/orchcli/notify.go:154-162`**
  - `NotifyPrime` returns `(false, nil)` when no orch strand is recorded, and the `Notify` seam drops the `posted` flag.
  - The retry loop therefore never retries, and nothing is logged beyond the Info line in `notifyPrime`.
  - The diff only retries refusals. It does not change how the receiver is resolved. The board's "reaches the orch by its real creel name" bullet depends on `OrchStrandOf`'s fixed-name fallback (`<shortname>:orch`).
- **F4, NIT, design, `internal/hibernateengine/verbs.go:192-197`**
  - A terminal request whose entry was already lifted now returns `Lifted` non-nil with an empty `Resumed`. The old result was nil.
  - I did not check how the CLI prints `Lifted`.

**Tool calls**
1. `grep` on `internal/hibernatecli/reconcile.go` for `Stop`, `Runs`, `Gone` and `Busy`: to find where `wake.Stop` and `wake.Runs` are populated, which my context lacked.
2. `Read` of `internal/hibernatecli/reconcile.go` lines 25-99: to confirm every stopped entry is appended to `Runs` when a shared branch stops.
