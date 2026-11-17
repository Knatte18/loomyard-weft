# Card 10 — Ways forward stop naming the retired adopt flags and pause

**Edit:**
- `internal/landingshed#ownerHaltReason`
- `internal/landingshed#TestFinalize_OwnerChecks`
- `internal/fabriccli#refuseForeignOwner`
- `internal/fabriccli#TestChangeAnchor`
- `internal/runproc#RoleAdopt`
- `internal/runproc#EndSubject`
- `internal/runproc/doc.go`
- `internal/lymdirs#AdoptDirName`
- `internal/pauserecord#AdoptCaller`
- `internal/pauserecord#HoldCaller`
- `internal/pauserecord/doc.go`

**Uses:**
- `internal/landingshed/doc.go`
- `internal/fabriccli/doc.go`
- `internal/lymdirs/doc.go`

**Intent:**
Refusals and docs outside adoptcli still name `--keep-local`, `--drop-local`, the adopt pause and the worker, which card 9 retired.

- `ownerHaltReason`: the adopt commands lose `--keep-local` (`lym adopt <parent subject> <subject>` and `lym adopt <subject>`), and the prime-owner case drops "which pauses the run here, so run lym tender resume ... and then lym loom resume" and the clause naming `--keep-local` and `--drop-local`: the adopt takes the run over, its check runs, and the run goes on by itself once the check is green.
- `refuseForeignOwner`: drops the `lym adopt <subject> --keep-local` alternative; the adopt's own takeover resets a weft to origin's tip, and its anchor-change refusal names `lym fabric anchor --set`.
- `RoleAdopt`'s comment and runproc's doc.go: the role an older lym's detached adopt worker registered under; a record of one is still spared at a subject's process end.
- `EndSubject`'s doc drops the adopt worker from the processes that serve the whole hub.
- `AdoptDirName`'s comment: the directory holds each subject's adopt check files and fix scratch, no record, lock or worker output.
- `AdoptCaller`, `HoldCaller` and pauserecord's doc.go: legacy callers an older `lym adopt` wrote; no code writes them now, the upgrade's first read turns a standing one into the Bolt model, and `CoverageOf` still reads one as held.

`TestFinalize_OwnerChecks` (tier 0) changes its `wantInReason` strings to the new commands and drops the `--keep-local`/`--drop-local` expectation; `TestChangeAnchor` (tier 0) drops the `--keep-local` expectation.
A `grep -rn -- '--keep-local\|--drop-local\|adopt --discard\|adopt worker' internal cmd contracts plugins` returns only retired-verb tests after this card.

**ImpactSummary:** refusal wording in landingshed and fabriccli with their two tests, and comments in runproc, lymdirs and pauserecord; no behaviour changes.

**Commit:** 10: Word the ways forward without the retired adopt flags and pause
