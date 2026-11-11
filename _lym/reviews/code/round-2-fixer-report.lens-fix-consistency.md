```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for agreement with its sibling texts.

No findings.

- The twin sentences that said `add` alone changes a beam project record now all name both verbs. These are `loom-directive-project-record.md:20`, `pawl-recipe.yaml:15`, `docs/overview.md:814`, `boardengine/runlock.go:67` and `boardengine/doc.go:472`.
- `internal/loomcli/doc.go:143` says `decision add <beam-slug>` resolves the beam's project record. The next sentences are about the caller rule, and `doc.go:147` already names both verbs. It is a description of what `add` does, not an "only way" claim, so the fix did not leave it stale.
- The new `writeFirstErr` and `writeAllErr` fields are used only by the two new replace rows, and the existing `restoreErr` semantics are unchanged.
- The new add refusal row matches `decisionWayLock`, which `decisionVerb` returns at `decision.go:885`.
- The commit messages follow the `<target>: fix <id> — …` format. The F1 commit also carries the shared fake fields and the `cmp` import that F2 uses, which is harmless.

Further tool calls:
1. `grep` over `*.go`, `*.md` and `*.yaml` for "decision add <slug>", "decision add <beam" and "one way the project": to find twins of the reworded sentences. The context did not hold them.
2. `sed -n 1255,1262p internal/loomcli/decision.go`: to see the store verb's lock refusal, a sibling of the add row. The context did not hold it.
3. `grep` for `writeFirstErr` and `writeAllErr` in the loomcli tests: to confirm every user of the new fake fields. The context did not hold the post-edit state.
4. `sed -n 139,147p internal/loomcli/doc.go`: to judge the `decision add <beam-slug>` hit at line 143. The context did not hold it.
