```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set — what the diff's changes force outside the lines they touch.

**Set per change**

1. `AppendDecisions` added, `AppendDecision` now delegates to it.
   - Callers of `AppendDecision`: only `append_test.go`, which still compiles against the unchanged signature.
   - `decision.go` now calls `AppendDecisions`.
   - `internal/discussionparser/doc.go:4` is not covered. It names only `AppendDecision`, "adds a post-Discussion design call", and says nothing of the batch write path. The package doc's inventory of write paths is now incomplete (finding 1).
   - Comments in `append.go` that say "AppendDecision writes/inserts" (lines 18, 34, 143, 178) and in `patternchanges.go` (184, 276) are still true, since `AppendDecision` is the one-entry case. Not a defect.
2. `decisionDeps.append` signature changed to take a slice.
   - The only constructor outside tests is `preRun` (`decision.go:728`), which was updated.
   - The test fake was updated and `lastAdded` is kept for existing tests.
   - Covered.
3. Success envelope gains `headings` and `pattern_changes`, and keeps `heading` and `pattern_change` for a one-entry call.
   - The grep found no consumer of these keys outside `decision.go` and its test. Covered.
   - One difference: the original always set `pattern_change` when the entry had one. Now it is set only when `len(added)==1`. That is the same behaviour for a one-entry call.
4. Notice text. A one-entry call's notice is unchanged, and its text still matches the format in the old test. A batch gets a short notice for ordinary receivers and an inline notice for Merriam.
   - The `decisionNoticeOpening` phrase is unchanged, so the parent-directive stencil (`parent-directive-parent.md:15`) and its test still hold.
   - `orch-template-role.md:111` says "the verb tells each live agent of the run that the record holds a new decision". Still true.
   - `darn-template-write.md:73` says the verb "types a line into your pane naming the new entry". This holds for a batch too, though the line now names several entries. No edit is needed.
   - Covered.
5. New `--from` flag and help.
   - The group `Long`, the `add` `Long`, the flag help, the file doc and `docs/overview.md:516` all name the batch form. Covered.
   - Fixed wording: the group `Long` still says "append decisions" correctly.
   - The `add` `Long` still says "`--title`, `--decision` and `--rationale` are required" without "unless `--from`". See finding 2.
6. Orch stencil line added at `orch-template-role.md:161`. It reads correctly against the code: `--from <file>`, a JSON array, one commit and one notice.
   - The arbiter stencil (`loom-template-arbiter.md:48`) and the bouncer stencil (`bouncer-template-escalation.md:48`) show the one-entry form only. The arbiter may pass `--from` since `--by arbiter` shares the code path, but the rubric asks only that stencils the diff changed are not weakened. Not a finding.
7. Unknown-key refusal. The decision record and acceptance criteria require an unknown key to be refused. `decisionEntries` (`decision.go:148-155`) builds a plain `json.NewDecoder` with no `DisallowUnknownFields`, and the grep for "unknown" found nothing in `decision.go` or `decision_test.go`. A misspelt key such as `pattern_enty` is silently dropped, so the entry is recorded without its PATTERN change. No test row reaches the refusal. This is the handler's and the code-correctness lens's territory, but the affected set (help text, `decisionWayFromFile`, which promises the keys) depends on it. I note it here and do not file it.

**Findings**

1. LOW, `consistency`: `internal/discussionparser/doc.go:4` documents only `AppendDecision` as the write path and omits `AppendDecisions`. The batch write is a changed public surface, and the package doc is where `CLAUDE.md` says a module's docs land in the same commit ("Docs land in the same commit"). Evidence: `doc.go:4` against `append.go:1-6` and `append.go:61`.
2. LOW, `consistency`: the `add` help in `decision.go:868-870` still says "`--title`, `--decision` and `--rationale` are required", and the usage line for the flags says nothing about `--from` replacing them there. The `--from` paragraph at lines 883-888 does say "in place of the per-entry flags". A reader of the required-flags sentence meets a contradiction until the later paragraph. Evidence: `decision.go:868-870` against `decision.go:883-884`.

**Tool calls**
1. A grep of the repo for `AppendDecision(s)`, `decision add` and `pattern_change`: the manifest lacked the callers outside the touched files.
2. `sed -n 1,12p internal/discussionparser/doc.go` plus a grep of `docs`, `PATTERN.md` and `pattern/` for `decision add`, and of `internal` for `"heading"`/`lastAdded`: to confirm the package doc's content and find any other consumers of the envelope keys.
