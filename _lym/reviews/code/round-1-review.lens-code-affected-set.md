```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. For each change, the set of things it touches outside its own lines.

**Per-change list**

1. Template default `cautious` → `wide`:
   - Covered: `internal/batcher/{config.go,registry.go,doc.go,template.yaml}`, `internal/websterengine/doc.go`, `docs/overview.md`, and the `config_test.go` / `verbs_test.go` assertions that read the default.
   - Not covered: `tools/sandbox/SANDBOX-WEBSTER-SUITE.md` still names `cautious` as the default (finding F-A).
   - My grep across md/yaml/go/tmpl/txt (research docs, `_lym`, `.lym` and test files excluded) found nothing else that calls `cautious` the default.
2. Former-default strip, `configsync.go` `formerTemplateValues["batcher"]["active"]` = `"cautious"`:
   - Covered: the `TestReconcileAll_MigratesToSparse` strip list in `configsync_test.go` gains the `wide` and `cautious.weights.orientation` entries.
   - The strip does not tell a hub's deliberate `cautious` from the old default. The commit message acknowledges this, so I raise no finding.
   - No other consumer of `formerTemplateValues` needs a change.
3. `cautious` profile now written out in full in the template: the retired-key migration reads `cautious` for `templateOrientation` (`config.go:246-256`), and the template still carries it, so that is covered.
4. `PATTERN.md` and `pattern/` have no diff in the range. The directive declares no PATTERN changes, so nothing is owed (`PATTERN-sparse-hub-base`: the strip drops only a template-equal unit, which fits).
5. Commit message: no `Pattern-Departure:` trailer, and none is needed. It also ends with `Co-Authored-By: Claude Opus 5.5`, not the Sonnet 5.5 line the session reminder asked for. That is the commit author's concern, not the diff's, so I don't flag it.

**Findings**

- **F-A, LOW, consistency** — `tools/sandbox/SANDBOX-WEBSTER-SUITE.md:5`
  - The summary says "the `cautious` cost profile by default", but the template default is now `wide`.
  - Evidence: line 5 of that file. The decision record says the comments that name `cautious` as the default follow "anywhere else that says so".
  - Fix: change it to "the `wide` cost profile by default".

**Tool calls (1)**
- One `grep` over the repo for `cautious` outside the files the diff touched, plus a check for `PATTERN.md` / `pattern/` changes in the range. My context only held the head of an earlier grep, which did not show the sandbox doc until now.
