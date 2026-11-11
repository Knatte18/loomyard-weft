```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens for the console-title format change. The list below gives, per change, the set I found and whether the diff covers it.

**Per change: set found, covered or not**
1. `consoletitle.Title` signature and the removal of `PrimeLabel`/`BeamLabel`/`OrchLabel`: no references to the removed labels are left anywhere. The callers are `handles.go` and `hubworkspace.go`, plus `smoke_drivingsurface_test.go`, and all are updated. Covered.
2. `IDEWindowTitle` and `TerminalWindowTitle` replaced by `ConsoleWindowTitle`: callers in `ideengine/spawn.go:295`, `battencli/wire.go:996` and `reedcli/window.go:62` are updated, and no other reference remains. Covered.
3. Title output, doc comments and prose: not fully covered.
   - **`internal/reedcli/window.go:80-83`**: the `lym reed window` help text still says the title is `"<hub shortname>: <P#>: <T#>: <slug>"`, `"<hub shortname>: orch"` for the prime and `"<hub shortname>: <P#>: orch"` for a beam.
   - **`internal/loomcli/sharedbootstrap.go:380`**: the doc comment still gives the example `ly: P3: T4: tapestry`.
   - **`internal/reedcli/window_test.go:27`**: the fixture `formedTitle = "ly: P3: T7: task"` still has the old shape. It is a stubbed value, so this is cosmetic.
   - **`docs/overview.md`, `consoletitle/doc.go`, `boardengine/doc.go`, `battenshed` doc and `template.yaml`, `loomcli/doc.go`, `fabricengine/doc.go` and `fabricengine/hubworkspace.go`**: all updated. Covered.
4. Workspace-file title `[ly] x workspace`: the integration tests in `fabricengine` and `ideengine` are updated. `docs/overview.md:592` describes it only as "names the hub shortname, then the repository or beam", which is still loosely true. Covered.
5. The Slack channel-name rule from the decision record: not covered.
   - **Record text:** it says "The Slack channel names in `slack-per-orch-activation` follow the same rule".
   - **What the diff does:** it does not touch `internal/slackengine/channelname.go` (`ChannelName(shortname, machine, slug)`) or `contracts/orchrole/role-slack.md`. `summary.md` defers this to another board entry.

**Findings**
- **MEDIUM, consistency.** `internal/reedcli/window.go:80-83`: the help text for `lym reed window` still describes the old `<hub shortname>: <P#>: <T#>: <slug>`, `: orch` and `<P#>: orch` forms. The default title now comes from `ConsoleWindowTitle`, so this user-visible text is wrong.
- **LOW, consistency.** `internal/loomcli/sharedbootstrap.go:380`: the doc comment example `ly: P3: T4: tapestry` should be `T4 [ly:P3] tapestry`.
- **NIT, consistency.** `internal/reedcli/window_test.go:27`: the stub `formedTitle` uses the old shape.
- **MEDIUM, scope.** The decision record's Goal says the Slack channel names follow the same orch rule. The diff changes no Slack code or contract, and `summary.md` defers it to the `slack-per-orch-activation` entry instead. I could not read that entry's body, because `lym board get` rejected the slug, so I can't tell whether the deferral is licensed. The parent should settle that.

**Tool calls**
1. Searched for callers of the changed `consoletitle` and `boardengine` title functions, to enumerate the affected set.
2. Searched for old-format title strings in Go, markdown and YAML, to find leftover references.
3. Searched for Slack files and `window.title`/`--title` mentions, to find other docs and users of the title.
4. Read `docs/overview.md:588-596`, the `role-slack.md` and `slackengine/channelname.go` hits, and `reedcli/window_test.go:20-35`, to check the workspace description, the Slack channel naming and the test fixture.
