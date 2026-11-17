```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer applied as proposed (F1, F3, F4, F5, F6) through the fix-proposal lens. I found no BLOCKING or MEDIUM findings and two LOW ones.

**LOW — F6, `declarationUnits` in `contracts/stencils/engineterms_test.go`**
- Summary: the declaration key still admits more than the one literal its reason describes, in three cases.
- Methods are keyed by `d.Name.Name` alone. Two methods of one name on different receivers in the same file, such as two `String()` methods, share one key. An entry for one admits the other.
- A grouped `var (a, b = "x", "y")` spec is keyed by `s.Names[0]` alone. A literal bound to `b` is admitted under `a`'s entry. Repeated `init` functions and `var _ = …` specs collide the same way.
- Entries `internal/heddlecli/cli.go#commandOf#system prompt` and `internal/heddleengine/overview.go#placeTitle#system prompt` admit every literal in the whole function that names the term. The reason text describes one help string, so a Codex-reaching literal added to `commandOf` later would pass.
- The key is narrower than the file-level one it replaces, but the narrowing stops at the declaration. The review's proposal named that boundary, so the gap comes from the proposal.

**LOW — F1, `internal/loomengine/discussiontable_test.go`, new "a codex chair…" subtest**
- Summary: the review's stated regression "passed an advisor's engine" is not caught by the row.
- The row uses `discussionTableConfig(0, false)`, so the table has no advisors, and the chair's engine is the only engine present. A producer that passed an advisor's engine instead of `chairChoice.Engine` has no advisor engine to pick, so the row cannot fail on that mistake.
- Evidence: `cfg := discussionTableConfig(0, false)`. With one or two advisors on `sonnet`/`opus` (Claude), the row would have caught that mistake.
- The review's own fix text only asked for "lead model-spec resolves to codex", which this row does. The gap is in the proposal.

**Checked, no case built**
- F1, begin-batch row: Master is on codex and Recovery stays on the fake engine, which carries Claude wording. If `beginDeps` read Recovery's choice or the default engine, the prompt would say "your system prompt" and the row would fail.
- F1, plan row: a producer that passed the default engine would give Claude wording, and `requireCodexForkWording` would fail on the missing `spawn_agent`/`wait_agent`.
- F3: the fixer wrote "Ten" and listed `background-command` as well as `fork-message`, matching the diff after the F2 ruling. The `send_message` addition matches the Codex `fork-message` copy.
- F4 and F5 are renames and a header sentence. Nothing reads them, and I found no breakage.

No further tool calls made.
