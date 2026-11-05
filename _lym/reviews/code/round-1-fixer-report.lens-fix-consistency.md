```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: do the fixes agree with their sibling docs, specs and registries?

Findings:

1. **MEDIUM** — `contracts/specs/raddle-proposal-spec.md:82`, a pinned spec the F1 change leaves stale.
   - The spec says Raddle-File runs the re-check "before its first step and again inside its board write".
   - It also says that in interactive mode "Raddle-File files only a proposal whose hash matches the recorded approval" (line 94).
   - After F1, a retry that finds the board already holding every filed entry skips `verify`.
   - That skips the check, the approval-hash match and the re-check.
   - It also skips the in-write `f.Check` in `FileRaddle`.
   - The behaviour is deliberate: otherwise `RecheckBoard` would report the run's own filed slugs as `slug-taken`.
   - The spec is a "Contract — pinned" doc and states no such exception.
   - Evidence: `internal/loomshed/raddlefile.go` `Call` (`!filed && !resumed`) and `internal/boardengine/raddlefile.go` `FileRaddle` (`raddleFilingHeld` branch before `f.Check`).
   - Suggested fix: add a sentence to the spec naming the half-done-filing exception.

2. **LOW** — `docs/overview.md` `_lym/usage/` row, a claim I could not confirm.
   - The row says the directory holds "the run's `run.json` and its `landed.json`".
   - `internal/lymdirs/dirs.go` says "one file per agent session, the run's identity and its landed diff size", without file names.
   - My grep found `run.json` and `landed.json` only in shuttle/webster code and in `internal/reportengine/cost_test.go` fixtures.
   - I did not find a non-test writer under `internal/shedrun/usage.go` or `lymdirs`.
   - The file names rest on the review's wording (F2's fix text) alone.
   - Suggested fix: confirm the names against the usage writer, or word the row like the `UsageDirName` comment.

Checked and consistent:
- Every registry and fixture that lists `RaddleFileDeps` seams now carries `AlreadyFiled`: `entries_raddle.go`, `entries_raddle_test.go`, `shedrecipe/fixture_test.go`, `envkit.go` and `battencli/wire_seams_test.go`. Only the first of these was a source of truth; the rest are test fixtures. The `loomcli` wiring sets it.
- The part and project body strings are built from the same `boardengine.RaddlePartBodyPrefix` and `RaddleProjectBodyPrefix` that `FileRaddle` and `HoldsRaddleFiling` match. The `boardengine` test helpers use them too.
- The `raddle-approve` text in `raddletable.go`, `cli.go` and `docs/overview.md` states no refusal condition, so the new notice refusal contradicts nothing.
- The verb doc comment and the `Long` text of the approve command are updated.
- The doc comment on `raddleProposeVerb` is updated.
- No other doc describes `Record-Write`'s support-log choice. The `StoredRecordSupportLog` references are in `recordwrite.go` and `discussionrecord.go`, and I did not read `discussionrecord.go`.

Tool calls:
1. Grepped `raddle-approve` in the spec, `raddletable.go`, `cli.go`, `wiring_test.go` and `overview.md`, plus the `run.json`/`landed.json` names and `StoredRecordSupportLog`. My context lacked these sibling documents.
2. Read spec lines 80-96 and grepped `UsageDirName` and the file names. My context lacked the spec's re-check wording and the usage writer.
