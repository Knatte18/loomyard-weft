# git-robustness: bounded git walks and reads, remote bounds, hub gc, batten window refusal

The run had one batch, `01-headcontains-bounded-walk`, covering all six cards.

The first attempt's `record-batch` returned `batch_failed`: the done-check found the Create target `internal/proc#ConfigureGroupKill` unresolved.
A backgrounded `lyx webster recover-batch 01` then finished with `status: done` at head `506ad41b38f1b30af67109c37575c560ab609585`.

The recovered batch covers:

- `HeadContains` stops its walk at a committer-time bound.
- Every go-git read in `internal/gitrepo` goes through one whole-operation retrying helper, with its PATTERN entry.
- The process-group kill moved into `internal/proc`, shared by `verifyrun` and `gitexec`.
- Every remote git command gets a deadline, a process-group kill and the low-speed environment, with timeouts classed as transport failures.
- Both hub stores set `gc.auto=0` and `maintenance.auto=false`, and a best-effort foreground gc runs after a pair's removal, with its PATTERN entry.
- `lyx batten run --window` runs the shared status check and refuses a done slug before opening a window.

The digest reported two deviations from the declared file-ops: `internal/proc/procgroup_linux.go` and `internal/proc/procgroup_windows.go`.
It also listed informational warnings for symbols touched outside the batch's target glyphs, mostly in `internal/gitrepo`, `internal/gitexec`, `internal/fabricengine`, `internal/verifyrun`, `internal/battencli` and `cmd/lyx`; none failed the batch.
