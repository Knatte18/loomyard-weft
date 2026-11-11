```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens, checking the round-1 change set against `doc.go`, the decision record and the code's own twins.

1. **LOW**: `internal/testcover/doc.go:65` and the decision record at D5 (`_lym/discussion/decision-record.md:52`) no longer match the removed-import rule in `compareGoSources`.
   - Both say an added or removed import is a miss, except an added import of a linked or standard-library package.
   - After F6, `internal/testcover/declcompare.go` skips the removed-import check when `after.present` is false. Deleting a file that imports anything is therefore no miss, and the doc and record don't say so.
   - The review's F6 asked for this change and the code is right. The `doc.go` bullet is the stale sibling, a one-line edit such as "an import removed from a file that remains".

2. **LOW**: `doc.go:63` says a non-Go file is a miss unless `inert_paths` matches it "and no go:embed pattern names it". It doesn't say that the importing package's own test files also count as embed sources, as F1 now makes them. The record at line 64 also says only "the linked package's Go files".
   - The linked-package half still holds, so this is an omission, not a contradiction.

3. **LOW**: `SelectorConfig.GoRoot` is new and is the only source of the standard-library check. `NewSelector` has no production caller yet (grep finds only `testselect_test.go`).
   - A future caller that leaves `GoRoot` empty would stat `src/<path>` relative to the working directory. Every standard-library import would then likely be a `MissNonBody`, which widens and is safe but silently disables the carve-out.
   - Nothing in the `GoRoot` comment says it is required. Optionally add "must be set" to the field comment.

No other twin was left stale:
- Both `otherFileMiss` call sites carry the new `ownPackage` argument.
- `embedPatterns` and both `embeds` accesses use `embedScope`.
- The `importAllowed` method call and definition agree.
- The F8 comment rewrites match the symbols they now name.
- The `ModulePath` check dropped from `importAllowed` is still covered, because `m/d` is not a directory under `GoRoot/src` in the test's fake root.
- The test's `AddedMethodNamedMax` row wants `MissMethodSet`, which agrees with record line 148, where the `max` case is a function.

Further tool calls (3 of 6):
- Grep of `doc.go`, `docs/overview.md` and the non-test sources for the standard-library, embed and `GoRoot` wording and for `otherFileMiss`, `importAllowed` and `embedPatterns` call sites; my context lacked those files.
- Read of `internal/testcover/doc.go` lines 50-85 to see the Miss list.
- Grep of `decision-record.md` for the D5 import and embed wording; my context lacked that file.
