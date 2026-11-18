```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change the diff makes, I looked for every caller, reader, doc and rule that describes it, and noted what the diff leaves wrong.

**Per change, the set found**

1. **Every push-lock taker passes the pause gate first (`fabricengine`, new `pushLockGateOp = "push-lock"`).**
   - `internal/fabricengine/doc.go:1241-1245` and `:1302-1306`: updated in the diff. Covered.
   - Doc comments on `coalescePush`, `coalesceFlagged`, `lockWeftForReplay`, `retryBoltPush`, `land` and `pullFirst`: updated. Covered.
   - `internal/pauserecord/doc.go:61-62` still says "An operation named remote-refs marks a lock hold." A `push-lock` operation now marks a lock hold too, so this sentence is stale. Not covered.
   - Readers of net-op names: tender lists net ops generically, and nothing matches on "remote-refs" outside `gitrepo` and `hold_test.go`. Nothing else is forced.
   - `statuscommit.Admit` (`internal/statuscommit/statuscommit.go:35-37`) nests admissions. `admissions` is a counter, so nesting is safe. Covered.
   - `PATTERN-gitrepo-client-boundary`, `PATTERN-daemon-wakeups` and `PATTERN-code-conventions` (the op name is one named constant): no conflict.
2. **The `pushDeferral` signature drops `pushed`; the `pushed` map is removed.**
   - Callers: `coalescePushSides` (updated) and `coalesce_test.go` (rewritten). No other caller. Covered.
3. **The board sync test now expects a plain covered sync to wait at the gate.**
   - `boardcli` tests and the `cli.go` sync help: the hidden `--deferrable` text is unchanged and still accurate. Covered.
4. **`Pause` and `Status` skip a pair with no reed session.**
   - `tenderengine/doc.go` and the `tendercli` pause and status `Long` text: updated.
   - The `StatusReport` JSON gains a `skipped` key. Its consumers are the JSON envelope only, which is additive. Covered.
   - `docs/overview.md:868` and `:874` describe pause and enumerate status's parts. Neither names the no-session skip or the new `skipped` part. Not covered.
   - `tendercli/cli.go:343` says "skipped lists each pair with no reed session". The prime is skipped too (`resume.go` `sessionStatuses`). Minor wording gap.
   - `contracts/orchrole/orch-role.md`: no text on this case. Nothing forced.
5. **`Revive` now skips a pair with no reed session**, a forced consequence of the shared `coveredSessions` signature.
   - The `ReviveReport.Skipped` field comment: updated.
   - The `tenderengine/doc.go` `# Revive` section (around `:114-138`) and the `tendercli` revive `Long` text (`cli.go:278-286`) do not mention the new skip. Not covered.
6. **Commit messages and trailers.** No `Pattern-Departure:` or `Loosened-Check:` trailers are present. No declared PATTERN changes exist, so none are owed.

**Findings**

1. LOW, consistency, `internal/pauserecord/doc.go:61`. The package doc still names `remote-refs` as the only operation that marks a lock hold, but `fabricengine` now marks `push-lock` holds through `Admit` too.
   Evidence: `internal/pauserecord/doc.go:61-62`, `internal/fabricengine/coalesce.go:38-41`.
2. LOW, consistency, `docs/overview.md:874` and `:868`. The tender overview lists every part of `lym tender status` but not the new `skipped` part. The pause sentence also omits that a pair with no reed session is skipped rather than failing.
   Evidence: `docs/overview.md:868`, `:874`; `internal/tenderengine/resume.go` `StatusReport.Skipped`.
3. LOW, consistency, `internal/tenderengine/doc.go` `# Revive` section (`:114-138`) and `internal/tendercli/cli.go:278-286`. Revive's new behaviour is documented only on the `ReviveReport.Skipped` field. The package doc's Revive section and the revive help text don't mention it, though the diff updated the matching pause and status docs. The behaviour: a pair with no reed session is reported under `skipped` instead of failing the verb.
   Evidence: `internal/tenderengine/resume.go` `Revive` (`coveredSessions` → `report.Skipped`), `internal/tendercli/cli.go:278-286`.
4. NIT, consistency, `internal/tendercli/cli.go:343`. The status help says `skipped` lists "each pair with no reed session". The prime is also listed there when it has none, as `sessionStatuses` and the `StatusReport.Skipped` comment ("the prime included") state.
   Evidence: `internal/tendercli/cli.go:343`, `internal/tenderengine/resume.go` `StatusReport.Skipped` comment.

**Tool calls**

1. Searched for docs and readers of the `remote-refs` op name, the old pushed-side wording, other `Admit` users, and the status JSON's readers. My context lacked the doc and consumer sites outside the diff.
2. Searched the `pauserecord` doc for its Admit and op text, and stencils and docs naming tender pause or status. My context lacked those docs.
3. Read the `docs/overview.md` tender section, the revive help text and the `tenderengine` Revive doc lines, to check whether the revive and status changes are documented there. My context lacked those spans.
