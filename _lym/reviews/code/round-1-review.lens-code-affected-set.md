```yaml
lens: code-affected-set
outcome: reported
```

**Task:** lens code-affected-set. I looked for everything outside the touched lines that the dotted-key `SetEntries` change forces to change.

**Per change: the set found, covered or not**
- **`SetEntries` dotted-key path**
  - The only production caller is `configengine.setEntries` (`internal/configengine/set.go:121`). It is reached only through `Set` when `spec.JSONEntries`, and only `runprofile.ConfigSpec` sets that (`internal/runprofile/runprofile.go:53`). Covered.
  - `configengine/set.go` and `configengine/doc.go` are updated in the diff. Covered.
- **Undotted refusal text, now "must be a JSON object, since … names a whole entry …"**
  - `configcli_test.go:1226` asserts the substring `"JSON object"`, which still matches.
  - `yamlengine/set_test.go` asserts the new text.
  - No other test, doc or stencil quotes the old `{"module": {"key": 1}}` text. Covered.
- **`Spec.JSONEntries` field comment** (`internal/configengine/layers.go:78`)
  - It still says a set writes "each pair as one whole top-level entry from a JSON object". It is the field's own documentation and the diff did not touch it. **Not covered (F1).**
- **Dot in a top-level name**
  - Before, `lym config profiles --set 'a.b={…}'` added a profile named `a.b`. Now it is read as a path.
  - `runprofile` refuses only an empty profile name (`runprofile.go:158`), so a dotted name is otherwise legal. Not covered (F2).
- **Docs**
  - `docs/overview.md`, `configcli/doc.go`, `runprofile/doc.go`, `yamlengine/doc.go` and `configengine/doc.go` are all updated.
  - `internal/runprofile/template.yaml:14` still names only the whole-profile form. The summary says this was left on purpose, to avoid changing seeded files, so I raise nothing.
  - `internal/batcher/doc.go:14` is a different module (`profiles.<name>.<key>` on batcher). Unaffected.
- **PATTERN entries and commit messages**
  - `PATTERN-config-layers` is not touched: the write still goes through `configengine`.
  - No `Pattern-Departure:` trailers are present, and no PATTERN changes are declared. Nothing to raise.
- **Post-write profile load check**
  - The summary claims a dotted key that leaves a profile in the wrong shape is refused and the file put back.
  - `TestDispatchSetProfiles` (`internal/configcli/configcli_test.go:1170`) has no dotted-key row, so nothing end to end pins that claim. Not covered (F3).

**Findings**

- **F1**
  - **Severity:** LOW
  - **Class:** consistency
  - **Location:** `internal/configengine/layers.go:78`
  - **Summary:** the `JSONEntries` field comment still says a set writes each pair as one whole top-level entry. It should also say that a dotted key sets one value below an entry.
  - **Evidence:** `layers.go:78` and `layers.go:100`, compared with the updated `configengine/set.go:48`.

- **F2**
  - **Severity:** LOW
  - **Class:** scope
  - **Location:** `internal/yamlengine/set.go:862`
  - **Summary:** for a `JSONEntries` module, any dot in a key now means a path. A profile name containing a dot (`v1.2`) can no longer be added or replaced by `--set`, and a JSON-object value at such a key would silently create a nested entry.
  - **Evidence:** `set.go:862`, with `runprofile.go:158` as the only name check.
  - **Note:** the board entry asks for dotted leaf keys, so the behavior is licensed. The change is that nothing documents or refuses the case.

- **F3**
  - **Severity:** LOW
  - **Class:** consistency
  - **Location:** `internal/configcli/configcli_test.go:1170`
  - **Summary:** the summary says a dotted key that leaves a profile in the wrong shape is refused by the post-write load and the file is restored. No `TestDispatchSetProfiles` row drives a dotted key through `lym config profiles --set`. The claim is unpinned, as is the board entry's own example (`docs.darn.loom.code_max_bounces=0`) at the CLI level.
  - **Evidence:** the rows read at `configcli_test.go:1170–1230` are all whole-entry keys.

**Tool calls (3 of 6)**
1. Repo-wide grep for "JSON object", "whole entry" and `JSONEntries`: to find stale prose, callers and tests that quote the changed text. The context lacked results outside `yamlengine`.
2. `grep` of `configengine/set_test.go` plus a read of `configcli_test.go:1165–1230`: to see whether existing set tests cover dotted keys or assert the old refusal text.
3. Grep for `WholeEntries` and profile-name validation in `runprofile`: to check whether a dotted profile name was previously legal.
