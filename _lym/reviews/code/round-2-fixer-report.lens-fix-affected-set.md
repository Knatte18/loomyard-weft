```yaml
lens: fix-affected-set
outcome: reported
```

Task: the fix-affected-set lens, rerunning each set-premised finding's search to check the fix covers every member.

**Finding left unchecked:** none. F3 and F5 are single-site fixes, so I did not search them.

**F1** (producers of the twill directive's engine and of begin-batch's engine): covered.
- `twilldirective.Directive` has two callers outside its own package, both in `internal/loomengine/twill.go` at lines 88 and 133. They are the plan and discussion lens directives, and the new codex rows cover both.
- `websterengine.BeginDeps{` is built in one place, `internal/webstercli/beginbatch.go:44`, and the new `TestBeginBatchCmd_PromptFollowsMasterEngine` covers it.

**F2** (Master stencil's backgrounded-command passages): the recover-batch, fix-verify and fix-plan passages are covered, with one gap.
- `webster-template-master.md` no longer has "backgrounded" for a Bash command, apart from line 92's "BACKGROUNDED agent", which describes the fork. `run_in_background` appears nowhere in the stencil.
- **LOW**, `internal/websterengine/verifygatereport.go:27`: the fix-verify way-forward text that Go sends Master says "run `%s` as a backgrounded Bash command, as the gate-failure section of your prompt says; then end your turn." On a Codex Master this contradicts the stencil's foreground Codex copy. It is production text and the same passage the decision moved, so a reader would notice it. Related stale Go comments are at `runlevel.go:769` and `doc.go:657`, `doc.go:888` and `doc.go:931`.
  - `verifygatereport_test.go:221` pins "the backgrounded fix-verify call", so a change touches that test too.
  - The parent's decision named the stencil passages. Whether it also covers this Go text is for the parent to call.
- **Out of scope, noted only:** the fork-completion passages (stencil lines 93–97, 105, 144, 188 and 230) still say "end your turn" and "completion notification". The Codex `fork-wait` copy has the parent wait in the same turn. The fix did not touch these lines, and they concern forks, not backgrounded Bash.

**F4** (stale fixture names): no `masterStencil`, `AllowlistedMasterLine` or `AnotherLineInTheMasterStencil` remains in any Go file. Covered.

**F6** (allowlist keys):
- No old-form key (file and term only) remains in `contracts/stencils/*_test.go`.
- The tier-0 gate passed with `RequireNoStale`, so every one of the five new keys matches a literal and the scan finds no unadmitted literal.
- Covered.

**Tool calls**
1. Searched `webster-template-master.md`, the other stencils, `masterStencil` and the old allowlist key form, because my context had only the pre-edit stencil and no repo-wide search.
2. Searched the callers of `twilldirective.Values`, `Directive` and `BeginDeps`, because my context had only the two producer sites the review named.
3. Searched Go production text for "backgrounded" under `internal/websterengine` and `internal/webstercli`, because my context had the stencil but not the Go text that mirrors it.
