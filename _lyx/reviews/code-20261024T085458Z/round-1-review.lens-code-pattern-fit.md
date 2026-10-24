```yaml
lens: code-pattern-fit
outcome: reported
```

Task: the code-pattern-fit lens, checking each rule against the changed files.

**Rules checked**
- `PATTERN-refusal-way-forward`: held for eight batch refusals, each with a `way forward:` text and a row in `TestDecisionVerb_Refusals`:
  - `--from` beside a per-entry flag
  - unreadable file
  - not an array, and data after the array
  - empty array
  - empty field
  - partial PATTERN form
  - repeated title
  - repeated PATTERN entry ID
- `PATTERN-refusal-way-forward`: broken for the unknown-key refusal. See finding 1.
- Record constraint "unknown keys are refused": broken. See finding 1.
- `PATTERN-tolerant-records`: not applicable, as the record says (caller input).
- `PATTERN-agent-name`: held. `decisionNoticeInline` classifies the strand through `agentname.Parse` and `MatchesRole`, and forms no name itself.
- Import of `websterengine`: `decision.go:51` is new, but `MerriamStrandRole` is an exported constant (`internal/websterengine/roles.go:39`). No rule in the PATTERN index forbids `loomcli` importing it.
- `PATTERN-parent-directive`: held. The opening phrase is unchanged and the directive stencil still quotes it (`parent-directive-parent.md:15`). `TestDecisionNoticeOpening_NamedByParentDirective` still covers that.
- `PATTERN-stencil-ownership` and `PATTERN-producer-pointer-rule`: held. The orch stencil gains one line that points at the verb's help and restates no format contract.
- `PATTERN-comment-line-breaks`: held on visual read, one sentence per line. The gate's `lint-comments` decides.
- `PATTERN-test-economy`: held.
  - `TestAppendDecisions_WritesBatchWholeOrNothing` covers a behavior no existing test covers (batch order, empty batch, later-entry error, later-entry finding).
  - The decision tests extend the existing refusals table and add one batch test.
- `PATTERN-test-speed`: held. The new tests call `t.Parallel()` and use the injected clock and fakes.
- `CLAUDE.md` docs rule: held. The verb's help, file doc, `docs/overview.md` and the orch stencil all name the batch form in the same commit.
- `CLAUDE.md` semantic line breaks in `.md`: held for the `docs/overview.md` and stencil additions, one sentence per line.
- `CLAUDE.md` Notes and Worktrees rules, and the declared PATTERN changes: nothing to check. The diff touches no `PATTERN.md` or `pattern/` file, and the directive lists no declared change in the part I was given.
- Commit messages: held. Neither commit carries a `Pattern-Departure:` trailer, and none is needed.
- Code-quality rule that a doc comment is self-contained: broken. See finding 2.

**Findings**

1. **MEDIUM · design** · `internal/loomcli/decision.go:148-155`
   - `decisionEntries` decodes with a plain `json.NewDecoder` and never calls `DisallowUnknownFields`, so a misspelt key such as `pattern_entery` is dropped silently.
   - The decision record (Constraints, and the acceptance criterion "an unknown key ... refused with a way forward") requires the refusal.
   - `grep -n "DisallowUnknown\|unknown"` finds nothing in `decision.go` or `decision_test.go`.
   - No `way forward:` text exists for it and no `TestDecisionVerb_Refusals` row reaches it. Both `PATTERN-refusal-way-forward` clauses fail for this refusal.
   - A misspelt `pattern_entry` would write the decision without its PATTERN bullet and still report success.

2. **LOW · consistency** · `internal/discussionparser/append.go:61-66`
   - The doc comment of `AppendDecisions` is defined in terms of another symbol: "the way AppendDecision adds one" and "as AppendDecision does".
   - The code-quality self-contained rule forbids that; the doc should state the restore behaviour directly.
   - `AppendDecisions` is also not a stdlib or self-reference exception.

**Tool calls (1 of 6)**
- `git diff` of `decision.go` filtered for imports, plus `grep` for `MerriamStrandRole` and the parent-directive stencil. My context did not show whether `websterengine` was a new import, whether the constant is exported, or whether the stencil still quotes the opening phrase.
