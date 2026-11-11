```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix's change set for stale twins and sibling disagreements (lens fix-consistency).

**Findings**

1. **LOW** — `internal/loomcli/decision.go:1808`, the `decision` group help. It says `"decision add <beam-slug>" records a change of the project's scope` and names no other verb. The three beam project-record texts F6 changed now name `add` or `replace`. `replace` is admitted on beam runs (`PATTERN-run-records.md:9` says so), so the group help is a twin the fix left as it was. `internal/loomcli/doc.go:143` (`` `decision add <beam-slug>` resolves the beam's project record ``) is a similar sentence. It only says what `add` resolves, so it is not wrong.

2. **LOW** — `contracts/specs/loom-discussion-spec.md:124` and `:126`. Both bullets say entries and bullets are "appended only by `lym loom decision add`". The F4 sentence added below them says the stored record "holds added entries once a motif run has made it", and `--record-file` folds that record. The two bullets are still true, since they concern appending. The new sentence adds no word on who may append to a stored record, and a reader could not tell whether `--record-file` may also write an added entry. The sentence's "not with `replace`" is also unsourced here. `replace` is refused on a raddle run whose filing is done, and it is allowed on a promoted-Sketch run before filing, where it does refresh the board copy. So "not with `replace`" is too absolute for the promoted-Sketch case.

No other contradiction found. The store help text in `decision.go:1890` matches `decisionStoreVerb` and the `boardCopyDue` doc at `:594`. The `replace` help names `--record-file`, and `PATTERN-run-records.md:9` already names `replace`.

**Tool calls**
1. A grep over `*.md`, `*.yaml` and `*.go` for "only add" and "no Discussion approval" phrasings. My context had only the files the fixer read, not every twin of the changed sentences.
2. A read of `decision.go:1800-1815`, to see the group help's wording around the beam sentence.
