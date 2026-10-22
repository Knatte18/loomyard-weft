---
round: 3
focus:
  - "In decision-record.md D3 and D4, a `lyx batten step` or `lyx batten run --window` is not registered with `runproc` and takes the run lock only after its preflight (`internal/battencli/arm.go` registerSeededRun, `internal/shedengine/run.go` run-lock try-acquire). Settle whether any batten that passed the cancel marker check unregistered and lock-free is refused or ended before the cancel releases its locks and removes the marker, and which gate the record names for it."
  - "In D3 step 3 and D4, the tackler marker is read at a point relative to step 5's End and step 7's scratch removal. Settle whether a tackler spawned by a still-live batten between the read and End is stopped, and whether the record states where the read sits."
  - "In D5 and D3, a `--remove` cancel re-run after step 6 (or on a run that never had a pair) reaches `RemovePair`, which returns an error wrapping `ErrPairNotFound` when nothing of the pair is left (`internal/fabricengine/remove.go`, `internal/pairteardown/teardown.go`). Settle whether the re-run still reaches the records removal, claim clear, lift, notice and marker removal."
  - "In D4 on Windows, `runproc.End` signals nothing and a record with an empty start time lives while its pid does (`internal/runproc/doc.go`). Settle whether the live-process refusal runs after the marker is written, and what bounds a reused pid for both the cancel marker's writer and a registered process, including the taskkill line D4 prints."
  - "In D2, D4 and D5, settle for each state a cancel can meet (task worktree gone with reed session or weft side left, kept pair whose entry the caller marked done, kept pair after `lyx fabric remove` leaving an unlanded task branch locally and on origin, a later `lyx batten run` on a kept pair) which steps run, which refusal fires, and whether the way forward the record names actually clears the slug."
---

Questions only; each names the code the record's claim rests on, so the reviewer judges the record's text against it.
