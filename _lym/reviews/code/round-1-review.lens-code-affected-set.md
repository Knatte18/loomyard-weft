```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I traced everything that reads or asserts it.

**Per change, the set found**

1. **Refusal text of `checkCwdAnchorGate`** (`internal/lymcwd/anchor.go:115-118`)
   - `lymcwd.Resolve` and `buildLocation` (`internal/lymcwd/lymcwd.go:218`): pass the error through. Covered.
   - `preflight.ResolveMode` (`internal/preflight/predicates.go:85-91`): returns the gated error unchanged, and creel `hubClient` (`internal/creelcli/cli.go:219-222`) returns it. Covered; creel gets the new text.
   - `driftcli.runCaptureChild` (`internal/driftcli/cli.go:258-262`): puts `err.Error()` into the lasting-fault envelope. Not covered: drift's halt reasons (`internal/driftengine/spawn.go:168`, `internal/driftengine/baseline.go:177`) embed the new text. See finding 1.
   - Tests that match substrings still hold: `internal/driftcli/anchor_integration_test.go:84`, `internal/driftengine/spawn_test.go:108,134,142`, `internal/driftengine/baseline_test.go:346,349`, `internal/lymcwd/lymcwd_test.go:50,361`, `internal/shuttlecli/resolve_verbs_test.go:42`, `internal/preflight/preflight_integration_test.go:653`.
   - Docs: no doc, stencil or spec quotes the old text. `internal/preflight/doc.go:43` and `pattern/PATTERN-cwd-resolution.md` describe the behaviour, not the text, and still hold.
   - `pattern/PATTERN-refusal-way-forward.md` and `contracts/specs/refusal-spec.md`: lymcwd is not a bound module. The new clause is a trailing `way forward:` clause and names no denied form, so it complies.
   - The shell-guard steer "never cd into the directory you already work in" (`internal/shellguard/rows.go:625`) is consistent with "cd &lt;anchored dir&gt;".
2. **New shipped heddle entry `lym-shell-directory`** (`internal/heddleengine/shipped.json:19-26`)
   - `ShippedEntriesArePinned` (`internal/heddleengine/store_test.go:185-195`) has a row for it. Covered.
   - `BothFilesHoldTheShippedLayerWithNoEntry` (`internal/heddleengine/render_test.go:494`) checks two headings with `Contains`, not the full list. It is not left wrong.
   - `RepositoryEntryUnderAShippedSlug…` uses `Ordered()[0]`. Unaffected.
   - `heddlecli` help names `lym-batching` only as an example. Unaffected.
   - `pattern/PATTERN-heddle-instructions.md` places lym's instructions in the shipped layer, which the diff complies with. No doc lists the shipped entries, apart from research data that is not a rule.
   - `pattern/PATTERN-cwd-resolution.md`: `ResolveWithin` accepts a cwd at or below the anchor, which the entry's wording contradicts. See finding 2.
3. **Commit trailers**: the `Root-Cause:` trailer names the message defect the diff changes. There is no `Pattern-Departure:` trailer, and the directive declares no PATTERN changes, so nothing is owed.
4. **`_lym/landing/summary.md`**: its "Blast radius: lymcwd" omits heddleengine's rendered agent files and drift's halt reasons. See finding 3.

**Findings**

1. **MEDIUM, design, `internal/lymcwd/anchor.go:115-118`.** Drift's capture-child refusal now carries a shell-`cd` cause and a "cd … and run the command again" way forward. Both are wrong for a child lym spawns, and they contradict drift's own halt reason.
   - lym starts the child in the anchored directory (`internal/driftengine/spawn.go:77`), so no agent shell or `cd` is involved.
   - `internal/driftcli/cli.go:260-262` passes `err.Error()` on as a lasting fault.
   - `internal/driftengine/spawn.go:168` and `internal/driftengine/baseline.go:177` wrap it in "lym loom resume cannot change this error; way forward: lym drift defer …". The halt reason then holds two `way forward:` clauses, and the embedded one says that re-running helps.
2. **LOW, consistency, `internal/heddleengine/shipped.json:21`.** The entry says a `lym` verb run after a `cd` "is refused". That is broader than the code: `ResolveWithin` verbs accept a subdirectory (`pattern/PATTERN-cwd-resolution.md`, `internal/lymcwd/lymcwd.go:141-160`), and `ResolveMode` treats a plain repo's subdirectory as standalone (`internal/preflight/predicates.go:85-93`).
3. **LOW, consistency, `_lym/landing/summary.md:28-31`.** The merge-danger blast radius names only lymcwd. The diff also changes the agents files heddleengine renders into every worktree, which includes the Codex `AGENTS.override.md`, and the halt-reason text drift writes (`internal/driftengine/spawn.go:168`).

**Tool calls**
- Searched for every reader of the refusal text and of the shipped-entry list. My context lacked the drift and render-test readers outside lymcwd.
- Read drift's halt-reason composition and the capture child's cwd. My context lacked how drift embeds the gated error.
- Read `render_test.go:480-510`. My context lacked what the shipped-heading assertion there checks.
