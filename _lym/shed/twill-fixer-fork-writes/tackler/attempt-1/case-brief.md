# Case brief: twill-fixer-fork-writes stopped at Seed-Child (producer)

## 1. The question the next judge must answer

Why did the records push of the prime (`main-weft`, in `/home/knatte/Code/loomyard-LYMHUB/loomyard-weft`) return nil with commits still unpushed, with no deferred record, so that `recordsSideNotLanded` reported the branch's entry as ended on origin? Is that a fabric defect (a wrong "ended" reading) and not a run defect, and should the run be re-launched or the fabric path fixed?

## 2. The facts, each with a pointer

- The stop envelope `/home/knatte/Code/loomyard-LYMHUB/loomyard/_lym/shed/twill-fixer-fork-writes/drive-reports/drive-envelope-20261118-110847-4c05.json`: `kind: producer`, `transient` empty, `history_length: 1`, `current_producer: Seed-Child`, `first_producer: Worktree-Create`, no prior resume.
- Stop error, trace `d8110d8471d7a88e`, `.trace.log` line 50 and `.stderr.log` line 1: `fabricengine: push records side at /home/knatte/Code/loomyard-LYMHUB/loomyard-weft: the entry of branch main-weft is ended on origin, so the branch was not pushed and is left to its teardown`.
- The same trace, `.trace.log` line 49: commit `c3801592add8f90b786ec438d56524840ca7f46d` created in `loomyard-weft`, after `.trace.log` line 45 spawned the detached push (pid 1541318).
- The message is produced at `internal/fabricengine/pushanchored.go:309`, in `recordsSideNotLanded` (lines 297-310). It is reached from `pushRecordsSide` (`pushanchored.go:213-220`) when the records `pushSide` returned nil and `HasUnpushed()` is still true. The fallback reads "ended" whenever no deferred record is found with `LastSeen` at or after `started` (`pushanchored.go:306-309`).
- Paths in `pushClassified` (`internal/fabricengine/pushclassified.go`) that return nil without a landed push: `checkOrigin` returns `proceed=false, err=nil` only for `view.ended` (lines 515-516); `deferral()` returns nil after writing a deferred record (lines 377-384), which the fallback should catch as a deferred error (`pushanchored.go:306`). A `view.unchecked` result proceeds to the push (lines 513-514). No log line in the trace says which path ran.
- Remote reads of `main-weft` on origin, in three calls: first `git ls-remote origin main-weft` returned `f70fe310148a5966cbccfd4abf7eda20a4839c14`; then `git ls-remote origin refs/heads/main-weft` returned `043880b99626143c15f13ef9f9d5c23915b9bcc2`; a full `ls-remote origin` listing returned `0b29395d82ecee92aeb6f3db0c0df8bd9e10e0b4` for `refs/heads/main-weft`. Origin's tip moved between my reads, so another writer is pushing `main-weft` concurrently.
- `git merge-base --is-ancestor c3801592add8f90b786ec438d56524840ca7f46d` succeeds against both `f70fe31` and `0b29395` (exit 0), so the stop's named commit is present on origin.
- Local `main-weft` in `loomyard-weft` is at `043880b...`, which equals the latest origin tip read.
- Design note `/home/knatte/Code/loomyard-LYMHUB/_bolt/design-records-push-main-weft-ended.md`: the same message hit `retro-stencil-fixes` and `shuttle-start-idle-and-prompt` on 2026-11-18. It says `main-weft` is never ended, that a rerun of `lym batten run <slug>` got the run through, and that the expected behaviour is that a prime records push never classifies `main-weft` as ended.
- Shed driver report `/home/knatte/Code/loomyard-LYMHUB/loomyard/_lym/shed/twill-fixer-fork-writes/drive-reports/drive-report-20261118-110847-97f1.md`: "no rule settles the stop: the tackler decides".
- Shed driver log `/home/knatte/Code/loomyard-LYMHUB/loomyard/.lym/logs/trace-20261118T100824Z-80dfbc13a3f5a518-1538907.log`, line 4: tackler spawned, deadline 11:18:47.

## 3. What I ruled out, and how

- Branch absent on origin: ruled out. Three `ls-remote` reads each return a line for `refs/heads/main-weft`. The `ended` wording in the message is not borne out by origin's state.
- Named commit absent: ruled out. `merge-base --is-ancestor` succeeds for `c3801592` against both remote tips read.
- Transient error: ruled out. `transient` is empty and the stop is `kind: producer`.
- Prior resume: ruled out. `history_length: 1`, no attempts in the drive report.
- A `goto` or a scope/design choice for the parent: not the cause. The failing row is `Seed-Child`, and the cause is in the fabric's classification, not in the run's plan.
- Whether a plain re-launch settles it: not settled by what I read. The design note says a rerun worked before, but the fallback message is misleading, and I could not tell from the logs which `pushClassified` path returned nil. A re-launch might pass or hit it again. That is why I did not choose `resume`.

## 4. My hypothesis (hypothesis, not established)

The records `pushClassified` call returned nil without landing and without writing a deferred record. The likely route is one that leaves no log line, such as `checkOrigin` returning `view.ended` for a `gone` view (`goneEntryEnded`) while origin's `main-weft` was transiently not found during the concurrent tip movement, or a deferred record whose `LastSeen` was not at or after `started`. The `recordsSideNotLanded` fallback then reports "ended", which is a misreading. The fix belongs in the fabric code (`pushanchored.go` and `pushclassified.go`), so a code-level judge should decide it. The run itself is not at fault.
